# 카오스 이론과 예측 (Part 2/2)

- **ChatGPT: 좋습니다. 그럼 바로 **fast/s...**
    
    좋습니다.그럼 바로 **fast/slow 역할 분리를 반영한 개선 코드**로 수정하겠습니다.
    
    이번 수정의 목표는 4가지입니다.
    
    1. slow가 fast 역할을 다 먹지 못하게 하기
    
    1. fast가 출력에 직접 기여하도록 강제하기
    
    1. fast/slow 기여도를 ablation으로 측정하기
    
    1. chaos 항이 실제로 쓰이는지도 같이 확인하기
    
    아래 코드는 이전 버전에서 핵심 부분만 바꾼 것이 아니라, 바로 실험 가능한 형태로 다시 정리한 버전입니다.
    
    # 핵심 변경점
    
    ## 1) readout 분리
    
    기존:
    [
    y_t = W[h_f; h_s]
    ]
    
    수정:
    [
    y_t = y_t^{(slow)} + y_t^{(fast)}
    ]
    [
    y_t^{(slow)} = W_s h_s,\quad y_t^{(fast)} = W_f h_f
    ]
    
    이렇게 하면 fast가 최소한 **출력 correction branch** 역할을 갖게 됩니다.
    
    ## 2) slow update rate 제한
    
    기존 slow gate는 자유롭게 커질 수 있었기 때문에 slow가 local dynamics까지 흡수할 수 있었습니다.
    
    수정:
    [
    \beta_t = \beta_{\max}\cdot \sigma(\cdot)
    ]
    로 두고, 예를 들어 `beta_max=0.08` 정도로 제한합니다.
    
    즉 slow는 진짜로 **느리게만** 움직이게 합니다.
    
    ## 3) fast/slow 사용량 모니터링
    
    추가 로그:
    
    • fast_contrib_norm
    
    • slow_contrib_norm
    
    • lam_mean
    
    이걸로 어떤 branch가 더 큰 출력을 만드는지 볼 수 있습니다.
    
    ## 4) validation ablation 추가
    
    평가 시 4개 모드를 비교합니다.
    
    • normal
    
    • fast off
    
    • slow off
    
    • chaos off
    
    이렇게 하면 정말로 fast가 쓰이는지 바로 확인할 수 있습니다.
    
    # 수정 코드
    
    아래는 **핵심 클래스 + loss + evaluate 함수**까지 포함한 버전입니다.
    
    ```python
    import math
    from dataclasses import dataclass
    from typing import Dict, Optional
    
    import torch
    import torch.nn as nn
    import torch.nn.functional as F
    
    # =========================================================
    # Config
    # =========================================================
    
    @dataclass
    class ChaosConfig:
        vocab_size: int
        d_model: int = 128
        d_fast: int = 128
        d_slow: int = 128
    
        # slow branch should move slowly
        slow_beta_max: float = 0.08
    
        # bounded chaos strength
        lambda_max: float = 0.25
    
        # regularization targets
        lambda_target: float = 0.10
        lambda_reg_weight: float = 0.01
    
        # encourage fast contribution not to collapse
        fast_balance_weight: float = 0.01
    
        # clipping
        state_clip: float = 3.0
    
    # =========================================================
    # Improved Chaos Cell
    # =========================================================
    
    class TinyChaosCellV2(nn.Module):
        """
        fast state: local lexical/syntactic dynamics
        slow state: long-context / discourse anchor
        """
    
        def __init__(self, cfg: ChaosConfig):
            super().__init__()
            self.cfg = cfg
    
            d_model = cfg.d_model
            d_fast = cfg.d_fast
            d_slow = cfg.d_slow
    
            self.fast_in = nn.Linear(d_fast + d_slow + d_model, d_fast)
            self.fast_gate = nn.Linear(d_fast + d_slow + d_model, d_fast)
    
            self.slow_in = nn.Linear(d_slow + d_fast + d_model, d_slow)
            self.slow_gate = nn.Linear(d_slow + d_fast + d_model, d_slow)
    
            self.ctrl = nn.Linear(d_fast + d_slow + d_model, 1)
    
            # separated readouts
            self.readout_fast = nn.Linear(d_fast, d_model)
            self.readout_slow = nn.Linear(d_slow, d_model)
    
        def chaos_nonlinearity(self, x: torch.Tensor, lam: torch.Tensor) -> torch.Tensor:
            # lam: [B,1]
            return torch.tanh(x) + lam * torch.sin(x)
    
        def forward(
            self,
            u_t: torch.Tensor,
            h_fast: torch.Tensor,
            h_slow: torch.Tensor,
            disable_fast: bool = False,
            disable_slow: bool = False,
            disable_chaos: bool = False,
        ):
            fast_cat = torch.cat([h_fast, h_slow, u_t], dim=-1)
            slow_cat = torch.cat([h_slow, h_fast, u_t], dim=-1)
            ctrl_cat = torch.cat([h_fast, h_slow, u_t], dim=-1)
    
            lam = self.cfg.lambda_max * torch.sigmoid(self.ctrl(ctrl_cat))  # [B,1]
            if disable_chaos:
                lam = torch.zeros_like(lam)
    
            # -------------------------
            # fast update: can react quickly
            # -------------------------
            fast_candidate = self.fast_in(fast_cat)
            fast_candidate = self.chaos_nonlinearity(fast_candidate, lam)
            fast_alpha = torch.sigmoid(self.fast_gate(fast_cat))
            new_h_fast = (1.0 - fast_alpha) * h_fast + fast_alpha * fast_candidate
    
            # -------------------------
            # slow update: intentionally limited
            # -------------------------
            slow_candidate = torch.tanh(self.slow_in(slow_cat))
            slow_beta = self.cfg.slow_beta_max * torch.sigmoid(self.slow_gate(slow_cat))
            new_h_slow = (1.0 - slow_beta) * h_slow + slow_beta * slow_candidate
    
            # optional ablation
            if disable_fast:
                new_h_fast = torch.zeros_like(new_h_fast)
            if disable_slow:
                new_h_slow = torch.zeros_like(new_h_slow)
    
            # stability
            new_h_fast = torch.clamp(new_h_fast, -self.cfg.state_clip, self.cfg.state_clip)
            new_h_slow = torch.clamp(new_h_slow, -self.cfg.state_clip, self.cfg.state_clip)
    
            # separated readout
            y_fast = self.readout_fast(new_h_fast)
            y_slow = self.readout_slow(new_h_slow)
            y_t = y_fast + y_slow
    
            aux = {
                "lam": lam,
                "fast_alpha_mean": fast_alpha.mean(dim=-1, keepdim=True),
                "slow_beta_mean": slow_beta.mean(dim=-1, keepdim=True),
                "fast_contrib_norm": y_fast.norm(dim=-1, keepdim=True),
                "slow_contrib_norm": y_slow.norm(dim=-1, keepdim=True),
                "h_fast_norm": new_h_fast.norm(dim=-1, keepdim=True),
                "h_slow_norm": new_h_slow.norm(dim=-1, keepdim=True),
            }
            return y_t, new_h_fast, new_h_slow, aux
    
    # =========================================================
    # Improved Chaos LM
    # =========================================================
    
    class TinyChaosLMV2(nn.Module):
        def __init__(self, cfg: ChaosConfig):
            super().__init__()
            self.cfg = cfg
    
            self.embed = nn.Embedding(cfg.vocab_size, cfg.d_model)
            self.cell = TinyChaosCellV2(cfg)
            self.lm_head = nn.Linear(cfg.d_model, cfg.vocab_size)
    
        def init_state(self, batch_size: int, device: torch.device):
            h_fast = torch.zeros(batch_size, self.cfg.d_fast, device=device)
            h_slow = torch.zeros(batch_size, self.cfg.d_slow, device=device)
            return h_fast, h_slow
    
        def forward(
            self,
            input_ids: torch.Tensor,
            disable_fast: bool = False,
            disable_slow: bool = False,
            disable_chaos: bool = False,
        ):
            B, T = input_ids.shape
            device = input_ids.device
    
            h_fast, h_slow = self.init_state(B, device)
    
            logits = []
            lam_hist = []
            fast_contrib_hist = []
            slow_contrib_hist = []
            h_fast_norm_hist = []
            h_slow_norm_hist = []
            fast_alpha_hist = []
            slow_beta_hist = []
    
            for t in range(T):
                u_t = self.embed(input_ids[:, t])
    
                y_t, h_fast, h_slow, aux = self.cell(
                    u_t,
                    h_fast,
                    h_slow,
                    disable_fast=disable_fast,
                    disable_slow=disable_slow,
                    disable_chaos=disable_chaos,
                )
                z_t = self.lm_head(y_t)
    
                logits.append(z_t.unsqueeze(1))
                lam_hist.append(aux["lam"].unsqueeze(1))
                fast_contrib_hist.append(aux["fast_contrib_norm"].unsqueeze(1))
                slow_contrib_hist.append(aux["slow_contrib_norm"].unsqueeze(1))
                h_fast_norm_hist.append(aux["h_fast_norm"].unsqueeze(1))
                h_slow_norm_hist.append(aux["h_slow_norm"].unsqueeze(1))
                fast_alpha_hist.append(aux["fast_alpha_mean"].unsqueeze(1))
                slow_beta_hist.append(aux["slow_beta_mean"].unsqueeze(1))
    
            logits = torch.cat(logits, dim=1)
            aux_all = {
                "lambda": torch.cat(lam_hist, dim=1),
                "fast_contrib_norm": torch.cat(fast_contrib_hist, dim=1),
                "slow_contrib_norm": torch.cat(slow_contrib_hist, dim=1),
                "h_fast_norm": torch.cat(h_fast_norm_hist, dim=1),
                "h_slow_norm": torch.cat(h_slow_norm_hist, dim=1),
                "fast_alpha_mean": torch.cat(fast_alpha_hist, dim=1),
                "slow_beta_mean": torch.cat(slow_beta_hist, dim=1),
            }
            return logits, aux_all
    
        @torch.no_grad()
        def generate(
            self,
            prompt_ids: torch.Tensor,
            max_new_tokens: int,
            temperature: float = 1.0,
            disable_fast: bool = False,
            disable_slow: bool = False,
            disable_chaos: bool = False,
        ):
            self.eval()
            device = prompt_ids.device
            B, T = prompt_ids.shape
    
            h_fast, h_slow = self.init_state(B, device)
    
            # warmup
            for t in range(T):
                u_t = self.embed(prompt_ids[:, t])
                _, h_fast, h_slow, _ = self.cell(
                    u_t,
                    h_fast,
                    h_slow,
                    disable_fast=disable_fast,
                    disable_slow=disable_slow,
                    disable_chaos=disable_chaos,
                )
    
            out = prompt_ids.clone()
            cur = prompt_ids[:, -1]
    
            for _ in range(max_new_tokens):
                u_t = self.embed(cur)
                y_t, h_fast, h_slow, _ = self.cell(
                    u_t,
                    h_fast,
                    h_slow,
                    disable_fast=disable_fast,
                    disable_slow=disable_slow,
                    disable_chaos=disable_chaos,
                )
                logits = self.lm_head(y_t) / max(temperature, 1e-5)
                probs = F.softmax(logits, dim=-1)
                cur = torch.multinomial(probs, 1).squeeze(1)
                out = torch.cat([out, cur.unsqueeze(1)], dim=1)
    
            return out
    
    # =========================================================
    # Loss
    # =========================================================
    
    def chaos_loss_fn_v2(
        logits: torch.Tensor,
        targets: torch.Tensor,
        aux: Dict[str, torch.Tensor],
        cfg: ChaosConfig,
    ):
        lm_loss = F.cross_entropy(
            logits.reshape(-1, logits.size(-1)),
            targets.reshape(-1)
        )
    
        lam_mean = aux["lambda"].mean()
        lam_reg = (lam_mean - cfg.lambda_target).pow(2)
    
        # encourage fast branch not to vanish completely
        fast_mean = aux["fast_contrib_norm"].mean()
        slow_mean = aux["slow_contrib_norm"].mean()
    
        # ratio near 1 would be too strong; use mild balancing
        balance = fast_mean / (slow_mean + 1e-6)
        fast_balance_reg = (balance - 0.3).pow(2)
    
        total = (
            lm_loss
            + cfg.lambda_reg_weight * lam_reg
            + cfg.fast_balance_weight * fast_balance_reg
        )
    
        stats = {
            "lm_loss": float(lm_loss.detach().cpu()),
            "lam_mean": float(lam_mean.detach().cpu()),
            "fast_contrib_mean": float(fast_mean.detach().cpu()),
            "slow_contrib_mean": float(slow_mean.detach().cpu()),
            "fast_slow_ratio": float(balance.detach().cpu()),
            "h_fast_norm_mean": float(aux["h_fast_norm"].mean().detach().cpu()),
            "h_slow_norm_mean": float(aux["h_slow_norm"].mean().detach().cpu()),
            "fast_alpha_mean": float(aux["fast_alpha_mean"].mean().detach().cpu()),
            "slow_beta_mean": float(aux["slow_beta_mean"].mean().detach().cpu()),
        }
        return total, stats
    
    # =========================================================
    # Evaluation with ablations
    # =========================================================
    
    @torch.no_grad()
    def evaluate_chaos_model_v2(model, loader, device: str, cfg: ChaosConfig):
        def run_mode(mode_name: str,
                     disable_fast: bool = False,
                     disable_slow: bool = False,
                     disable_chaos: bool = False):
            model.eval()
            total_loss = 0.0
            total_tokens = 0
    
            agg = {
                "lam_mean": 0.0,
                "fast_contrib_mean": 0.0,
                "slow_contrib_mean": 0.0,
                "fast_slow_ratio": 0.0,
                "h_fast_norm_mean": 0.0,
                "h_slow_norm_mean": 0.0,
                "fast_alpha_mean": 0.0,
                "slow_beta_mean": 0.0,
            }
            n_batches = 0
    
            for x, y in loader:
                x = x.to(device)
                y = y.to(device)
    
                logits, aux = model(
                    x,
                    disable_fast=disable_fast,
                    disable_slow=disable_slow,
                    disable_chaos=disable_chaos,
                )
                loss, stats = chaos_loss_fn_v2(logits, y, aux, cfg)
    
                n_tokens = x.numel()
                total_loss += loss.item() * n_tokens
                total_tokens += n_tokens
                n_batches += 1
    
                for k in agg:
                    agg[k] += stats[k]
    
            avg_loss = total_loss / max(total_tokens, 1)
            ppl = math.exp(min(avg_loss, 20))
    
            result = {
                "mode": mode_name,
                "loss": avg_loss,
                "ppl": ppl,
            }
            for k in agg:
                result[k] = agg[k] / max(n_batches, 1)
    
            return result
    
        results = {
            "normal": run_mode("normal"),
            "fast_off": run_mode("fast_off", disable_fast=True),
            "slow_off": run_mode("slow_off", disable_slow=True),
            "chaos_off": run_mode("chaos_off", disable_chaos=True),
        }
        return results
    ```
    
    # 기존 train loop에 붙이는 방법
    
    이제 학습 쪽에서는 기존 `TinyChaosLM` 대신 `TinyChaosLMV2`를 쓰고, loss도 `chaos_loss_fn_v2`로 바꾸면 됩니다.
    
    예를 들면:
    
    ```python
    cfg_model = ChaosConfig(
        vocab_size=tokenizer.vocab_size,
        d_model=128,
        d_fast=128,
        d_slow=128,
        slow_beta_max=0.08,
        lambda_max=0.25,
        lambda_target=0.10,
        lambda_reg_weight=0.01,
        fast_balance_weight=0.01,
        state_clip=3.0,
    )
    
    model = TinyChaosLMV2(cfg_model).to(device)
    optimizer = torch.optim.AdamW(model.parameters(), lr=3e-4, weight_decay=1e-4)
    ```
    
    학습 스텝:
    
    ```python
    model.train()
    for step, (x, y) in enumerate(train_loader, start=1):
        x = x.to(device)
        y = y.to(device)
    
        optimizer.zero_grad()
        logits, aux = model(x)
        loss, stats = chaos_loss_fn_v2(logits, y, aux, cfg_model)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        optimizer.step()
    
        if step % 100 == 0:
            print(
                f"step={step} "
                f"loss={loss.item():.4f} "
                f"lam={stats['lam_mean']:.4f} "
                f"fast={stats['fast_contrib_mean']:.4f} "
                f"slow={stats['slow_contrib_mean']:.4f} "
                f"ratio={stats['fast_slow_ratio']:.4f} "
                f"fa={stats['fast_alpha_mean']:.4f} "
                f"sb={stats['slow_beta_mean']:.4f}"
            )
    ```
    
    검증:
    
    ```python
    results = evaluate_chaos_model_v2(model, valid_loader, device, cfg_model)
    
    for name, r in results.items():
        print(
            f"[{name}] "
            f"loss={r['loss']:.4f} ppl={r['ppl']:.2f} "
            f"lam={r['lam_mean']:.4f} "
            f"fast={r['fast_contrib_mean']:.4f} "
            f"slow={r['slow_contrib_mean']:.4f} "
            f"ratio={r['fast_slow_ratio']:.4f}"
        )
    ```
    
    # 결과를 어떻게 해석해야 하나
    
    이제 가장 중요한 부분입니다.
    
    ## 1. fast가 실제로 쓰이는 경우
    
    정상 학습 후:
    
    • normal
    
    • fast_off
    
    • fast_contrib_mean / slow_contrib_mean
    
    • fast_alpha_mean
    
    이러면 fast state가 실제 기여하고 있다고 볼 수 있습니다.
    
    ## 2. 여전히 fast가 죽은 경우
    
    아래 패턴이면 fast collapse 가능성이 큽니다.
    
    • fast_off
    
    • fast_contrib_mean << slow_contrib_mean
    
    • fast_slow_ratio
    
    • fast_alpha_mean
    
    이 경우는 fast branch가 거의 무시된 것입니다.
    
    ## 3. slow가 너무 약한 경우
    
    반대로 slow를 너무 느리게 만들면:
    
    • slow_off
    
    • long-range coherence가 약함
    
    • 생성 초반만 맞고 이후 topic drift가 심함
    
    이 경우 `slow_beta_max`를 약간 올려야 합니다. 예: 0.08 → 0.12
    
    ## 4. chaos가 실제 도움을 주는 경우
    
    `chaos_off`와 `normal` 비교에서:
    
    • normal
    
    • generation diversity가 더 좋음
    
    • 반복 루프가 줄어듦
    
    이면 chaos-like nonlinearity가 의미가 있습니다.
    
    반대로 `chaos_off`가 더 좋으면 현재의 chaos 항은 오히려 잡음일 가능성이 큽니다.
    
    # 추천 하이퍼파라미터 탐색 순서
    
    처음에는 아래 3개만 바꾸면 됩니다.
    
    ## 1. `slow_beta_max`
    
    • 0.05
    
    • 0.08
    
    • 0.12
    
    ## 2. `lambda_max`
    
    • 0.0
    
    • 0.10
    
    • 0.25
    
    ## 3. `fast_balance_weight`
    
    • 0.0
    
    • 0.005
    
    • 0.01
    
    이걸 돌리면 구조가 꽤 빨리 보입니다.
    
    # 지금 수정의 의미
    
    이번 수정으로 모델은 최소한 다음 성질을 갖게 됩니다.
    
    • fast/slow가 
    
    • slow는 진짜로 천천히 움직이도록 제한
    
    • fast는 출력에 직접 관여
    
    • ablation으로 실제 기여도 확인 가능
    
    즉, 이제부터는 단순히 “아이디어가 예쁘다”가 아니라**“fast state가 정말 필요한가?”**를 실험적으로 검증할 수 있는 단계로 들어갑니다.
    
    # 다음 단계에서 가장 좋은 일
    
    이제는 코드 자체보다 **실험 관측 장치**가 더 중요합니다.다음으로는 이 버전에
    
    • epoch별 ablation 결과 저장
    
    • fast/slow/lam 히스토리 플롯
    
    • generation sample 자동 저장
    
    을 넣는 것이 가장 좋습니다.
    
    원하시면 다음 답변에서 바로 **전체 학습 스크립트를 이 V2 구조로 통합한 완성본**으로 정리해드리겠습니다.