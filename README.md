markdown
# Emotion-Tabu Edge Biped: Autonomous Night-Escort Robot Architecture


![Status](https://shields.io)
![Platform](https://shields.io)
![Hardware](https://shields.io)

> **⚠️ Project Status / プロジェクトの位置づけについて**
> **EN:** This repository is a **Concept Proposal and Architectural Design (Pre-Implementation / Planning Phase)**. While the mathematical formulation and initial draft designs for a 2D simulation have been outlined, **actual implementation and testing are upcoming steps**. I would be very happy if you find this idea interesting!
>
> **JA:** 本リポジトリは、現在進行形の**コンセプト提案（ジャストアイデア）およびアーキテクチャ設計段階（これから実装・検証を進めるフェーズ）**です。数理モデルやPC上の2Dシミュレーションに向けた設計メモは用意していますが、**実際の実装・動作確認はこれから取り組む段階**です。このアイデアに少しでも興味を持っていただけたら嬉しいです。

---

## 1. Background & Origin / 背景と発想の原点

**EN:**
This project is a direct physical expansion of my previous work: a **Japanese Female Bartender Conversational Droid** running offline on Android (Termux). In that dialogue system, I discovered that combining a lightweight LLM (`Gemma 2B`) strictly for *domain classification* with an *Eliza-style dictionary + Tabu Search* produces remarkably human-like, non-repetitive, and context-aware conversations without heavy cloud dependencies.

**Core Idea:** *"Why not apply the exact same 'Tabu Search + Lightweight Rule/Policy' architecture to physical bipedal locomotion?"*
By bridging the bartender droid's conversational brain with a lightweight physical control engine, the robot aims to **escort a drunk customer safely home through dark night streets**—asking passersby for directions, reading human facial reactions ("That's weird / NG"), and adapting its walking style in real time.

**JA:**
本プロジェクトは、私が以前に開発・公開した**「女性バーテンダーとのオフライン会話ドロイド」の発想を、身体（2足歩行制御）へと直接拡張しようという構想**です。前作の対話システムでは、軽量LLM（`Gemma 2B`）に「会話ドメインの判定」のみを担当させ、実際の応答生成を「Eliza的辞書 ＋ タブーサーチ（同じ返答や不自然な返答の回避）」で行うことで、クラウドに頼らず人間らしい間合いと多様性を実現しました。

**核心となるアイデア：** *「会話で成功した『タブーサーチ × 超軽量モデル』の仕組みを、そのまま2足歩行の歩き方（歩容）制御に拡張できないか？」*
バーテンダーの対話脳と軽量の身体制御エンジンを連動させることで、通りすがりの人に道を尋ね、相手の表情（「それ変ダヨ」という困惑顔）や路面の状況を読み取りながら、リアルタイムに歩き方や態度を変化させる仕組みをこれから実装していきます。

# Emotion-Tabu Edge Biped

## 🇯🇵 日本語版 (Japanese)

### 超軽量エッジAIによる自律2足歩行アーキテクチャ

#### 概要
本プロジェクトは、高価なGPUや巨大な深層強化学習（DRL）モデルに頼らず、Android端末（Termux）などのエッジデバイス上でミリ秒推論を実現する、自律2足歩行ロボットのための軽量な数理アーキテクチャです。
「脳（全体の方針）」「神経（局所の異常管理）」「筋肉（運動実行）」の3層構造を人工ポテンシャル場とベクトル演算のみで統合し、対話しながらの自律帰還、歩幅の動的学習、痛みをかばう代償動作、そして「絶対に過ちを繰り返さない」防衛本能をプログラムの複雑な分岐なしで実現します。
（※注: ロボットの物理的なバランス維持やZMP補償は、下位のハードウェア制御に依存する役割分担を前提としています）

#### 主要なシステム機能
* **対話を通じた自律帰還:** 音声コマンド（「帰ろう」など）を受信すると目的地をセットし、「一緒にお家に帰りましょう」と対話しながら、道中の障害物を自律的に迂回・回避して目的地へ向かいます。
* **人間に道を尋ねるヘルプ要請:** 障害物に囲まれてスタックした状態（移動分散の低下）を自動検知すると、自発的に「どっちに行けばいいか教えて？」と周囲の人間に助けを求めます。人間の指示（「右」「左」「〇〇ビル」）を受けて新たなウェイポイントを追加し、自力で脱出ルートを再生成します。
* **絶対的緊急停止の割込発話 (Interrupt):** 雑談対話中であっても、タスク脳が致命的危険（「危ない」という声や、LiDAR/Visionでの異常接近）を察知した瞬間にトークを強制キルし、「危ない！止まります！」という警告とともに即時停止処理を実行します。
* **マルチモーダル非同期処理と歩幅学習:** 音声（STT）、視覚（Vision）、足元（LiDAR/IMU）のセンサー入力を独立スレッドで並行処理します。さらにIMUの上下動ピークとLiDARの移動距離から、現在の「歩幅」と「速度」をリアルタイムに自動校正します。

#### アーキテクチャと数式モデル

**1. 脳 (Brain): Dual Tabu Potential Field**
目標への引力と、2種類の失敗記憶（一時タブー・永久タブー）からの斥力を合成し、ロボットの意思となる「気質ベクトル $\mathbf{e}_t$」を毎ステップ更新します。

$$ \mathbf{e}_{t+1} = \mathbf{e}_{t} - \alpha \nabla_{\mathbf{e}} \left( U_{\text{goal}}(\mathbf{e}_{t}) + U_{\text{human}}(\mathbf{e}_{t}) + U_{\text{fatal}}(\mathbf{e}_{t}) \right) $$

* **目標への引力（2次ポテンシャル）:**
理想の目的地 $\mathbf{e}_{\text{target}}$ へ引き寄せるバネの力。スタック時（局所解検知時）はゲイン $k_{\text{goal}}$ を動的に下げて脱出を優先させます。

$$ U_{\text{goal}}(\mathbf{e}_{t}) = \frac{1}{2} k_{\text{goal}} \|\mathbf{e}_{t} - \mathbf{e}_{\text{target}}\|^2 $$

* **一時タブーからの斥力（時間減衰ガウス関数）:**
人間の訂正などソフトな失敗記憶 $\mathcal{H}_{\text{human}}$ から遠ざける力。時間経過で忘却します。実装では肩幅や速度を反映した側方スリップ力も合成されます。

$$ U_{\text{human}}(\mathbf{e}_{t}) = \sum_{j \in \mathcal{H}_{\text{human}}} A_{\text{soft}} e^{-\gamma (t - t_j)} \exp\left( -\frac{\|\mathbf{e}_{t} - \mathbf{e}_j\|^2}{2\sigma_{\text{soft}}^2} \right) $$

* **永久タブーからの斥力（永続ガウス関数）:**
転倒や異常接近など致命的な失敗記憶 $\mathcal{H}_{\text{fatal}}$ に対する絶対障壁。減衰しません。ハードウェアへの負担を減らすため、係数 $A_{\text{fatal}}$ は移動速度に応じて動的に増幅されます。

$$ U_{\text{fatal}}(\mathbf{e}_{t}) = \sum_{k \in \mathcal{H}_{\text{fatal}}} A_{\text{fatal}} \exp\left( -\frac{\|\mathbf{e}_{t} - \mathbf{e}_k\|^2}{2\sigma_{\text{fatal}}^2} \right) $$

* **すり抜け防止 (Kinematic Clipping):**
斥力が無限大に発散しても障害物の向こう側へワープ（オーバーシュート）しないよう、1回の更新量は現在の歩幅から算出される最大ステップ制限 $max\_step$ で物理的に切り詰められます。

$$ \mathbf{e}_{t+1} \leftarrow \mathbf{e}_t + \text{clip}(\Delta \mathbf{e}, -max\_step, max\_step) $$

* **脳の可塑性（メタ・タブーサーチ）:**
移動の分散が閾値を下回った場合を「スタック」と検知し、一時タブーの減衰率 $\gamma$ を加速させ、自律的にパラメータを書き換えて局所解から脱出します。

$$ \text{Stuck}(t) = \begin{cases} 1 & \text{if } \text{Var}(\mathbf{e}_{t-\tau:t}) < \epsilon \\ 0 & \text{otherwise} \end{cases} $$

$$ \gamma(t) = \gamma_{\text{base}} + \Delta \gamma \cdot \text{Stuck}(t) $$

**2. 神経 (Nerves): 局所関節の動的マスキング**
関節ごとの異常（過熱等 $p_{j,t}$）を監視し、危険な関節のみアグレッシブなポリシーの重みを0にして、安全ポリシー $\delta_{i, \text{safe}}$ へと動的に切り替えます。

$$ w_{i,j}(\mathbf{e}_{t}, p_{j,t}) = \text{Softmax}(\mathbf{e}_{t})_{i} \times (1 - \text{Mask}(p_{j,t})) + \delta_{i, \text{safe}} \times \text{Mask}(p_{j,t}) $$

**3. 筋肉 (Muscles): TinyMLPアンサンブル**
事前学習済みの極小ポリシー $\pi_i$（筋肉）に対し、神経から渡された関節ごとの重みベクトル $\mathbf{w}_i$ を要素ごとの掛け算（アダマール積 $\circ$）でブレンドし、最終的な関節指令 $\mathbf{u}_t$ を出力します。

$$ \mathbf{u}_{t} = \sum_{i=1}^{N} \left( \mathbf{w}_{i}(\mathbf{e}_{t}, \mathbf{p}_{t}) \circ \pi_{i}(\mathbf{s}_{t}) \right) $$

#### 優位性 (Pros)
* **圧倒的なエッジ性能と安定性:** 重い行列最適化を完全に排除。`np.clip` によるKinematic Clipping（すり抜け防止）や永久タブーのハードリミット実装により、スマホCPUでもミリ秒推論が破綻せず安定稼働します。
* **破滅的忘却のない即時適応:** 致命的な失敗は「永久タブーの絶対障壁」として空間に後乗せされます。元の歩行ポリシーを壊すことなく「1秒前の失敗を二度と繰り返さない」安全行動を即座に獲得します。
* **代償動作の自然発現:** 動的マスキングにより、複雑な条件分岐（if文）を記述することなく「痛い関節だけをかばって歩く」という生物的な振る舞いが数式上から自然発現します。
* **動的バイアスによる環境適応:** スタック時の引力緩和や、移動速度に応じた斥力ゲインの増幅など、状況に応じてポテンシャル場を歪ませることで、局所解からの脱出と物理的な安全マージンの確保を実現します。

#### 課題と限界 (Cons / Limitations)
* **ZMPの無視と転倒リスク:** ソフトウェア（脳）側での急な回避や代償動作は数式上滑らかにブレンドされますが、物理世界では急激な重心移動を伴います。下位のハードウェア（PID/ZMP制御）がこれを吸収しきれない場合、転倒の引き金となるリスクがあります。
* **ヒューリスティックなパラメータへの依存:** 引力・斥力ゲインなど、手動設定に依存するパラメータが多く存在します。初期値が実機の物理特性と乖離しすぎていると、微振動（チャタリング）を誘発します。
* **長期稼働時のタブーメモリ統合:** 永久タブーには配列上限を設けて計算破綻を防いでいますが、押し出された古い記憶を単に破棄するのは最適ではありません。将来的にはガウス混合モデル（GMM）等を用いて近接記憶を空間的に統合・圧縮する仕組みが必要です。

---

## 🇬🇧 英語版 (English)

### Ultra-Lightweight Edge AI Architecture for Autonomous Bipedal Robots

#### Overview
This project introduces a lightweight mathematical architecture for autonomous bipedal robots, designed to run millisecond inference on edge devices (like Android/Termux) without relying on expensive GPUs or massive Deep Reinforcement Learning (DRL) models.
It integrates a 3-layer structure—"Brain", "Nerves", and "Muscles"—using artificial potential fields and vector math. It features autonomous return-to-home with dialogue, dynamic stride learning, compensatory movements (limping), and an absolute defense mechanism against fatal errors, all achieved without complex branching logic.
*(Note: This architecture assumes that the low-level hardware handles physical balance and Zero Moment Point (ZMP) compensation.)*

#### Key Functional Features
* **Autonomous Return with Dialogue:** Upon receiving a voice command (e.g., "Let's go home"), the robot sets the destination waypoint and autonomously navigates while avoiding obstacles, responding naturally like a companion ("Yes! Let's go home. Follow me.").
* **Help Requests via Meta-Tabu Search:** If the robot detects it is stuck surrounded by obstacles (low movement variance), it autonomously asks nearby humans for help ("Can you tell me which way to go?"). It then listens for directions (e.g., "Right", "Left", or landmarks) to dynamically generate new escape routes.
* **Emergency Speech Interruptions:** Even in the middle of a casual chat, if the "Task Brain" detects a fatal danger via voice ("Watch out!") or LiDAR/Vision, it forcibly kills the ongoing conversation and executes an immediate stop while uttering a warning.
* **Multimodal Processing & Stride Learning:** Voice (STT), Vision, and LiDAR/IMU inputs are processed in parallel asynchronous threads. The robot automatically recalibrates its "stride" and "speed" in real-time using the distance delta from LiDAR and vertical acceleration peaks from the IMU.

#### Architecture and Mathematical Models

**1. Brain: Dual Tabu Potential Field**
Updates the robot's "temperament vector $\mathbf{e}_t$" every step by calculating the resultant force of attraction to a goal and repulsion from two types of failure memories (Soft and Fatal Taboos).

$$ \mathbf{e}_{t+1} = \mathbf{e}_{t} - \alpha \nabla_{\mathbf{e}} \left( U_{\text{goal}}(\mathbf{e}_{t}) + U_{\text{human}}(\mathbf{e}_{t}) + U_{\text{fatal}}(\mathbf{e}_{t}) \right) $$

* **Attraction to the Goal (Quadratic Potential):**
A spring-like force pulling toward the ideal destination $\mathbf{e}_{\text{target}}$. When a stuck state is detected, the gain $k_{\text{goal}}$ is dynamically reduced to prioritize escape.

$$ U_{\text{goal}}(\mathbf{e}_{t}) = \frac{1}{2} k_{\text{goal}} \|\mathbf{e}_{t} - \mathbf{e}_{\text{target}}\|^2 $$

* **Repulsion from Soft Taboos (Time-decaying Gaussian):**
Force to avoid soft failure memories $\mathcal{H}_{\text{human}}$ (e.g., human corrections). It decays over time. The implementation synthesizes a lateral slip force based on stride and speed.

$$ U_{\text{human}}(\mathbf{e}_{t}) = \sum_{j \in \mathcal{H}_{\text{human}}} A_{\text{soft}} e^{-\gamma (t - t_j)} \exp\left( -\frac{\|\mathbf{e}_{t} - \mathbf{e}_j\|^2}{2\sigma_{\text{soft}}^2} \right) $$

* **Repulsion from Fatal Taboos (Permanent Gaussian):**
An absolute barrier against fatal failures $\mathcal{H}_{\text{fatal}}$. It does not decay. The coefficient $A_{\text{fatal}}$ dynamically amplifies based on the robot's moving speed to reduce the physical burden on the hardware during deceleration.

$$ U_{\text{fatal}}(\mathbf{e}_{t}) = \sum_{k \in \mathcal{H}_{\text{fatal}}} A_{\text{fatal}} \exp\left( -\frac{\|\mathbf{e}_{t} - \mathbf{e}_k\|^2}{2\sigma_{\text{fatal}}^2} \right) $$

* **Kinematic Clipping (Overshoot Prevention):**
Even if the repulsion diverges to infinity, the actual vector update is physically clamped by $max\_step$ (derived from the current learned stride) to prevent warping through obstacles.

$$ \mathbf{e}_{t+1} \leftarrow \mathbf{e}_t + \text{clip}(\Delta \mathbf{e}, -max\_step, max\_step) $$

* **Brain Plasticity (Meta-Tabu Search):**
Detects local optima (stuck states) when movement variance drops, and autonomously accelerates the decay rate $\gamma$ of soft taboos to escape.

$$ \text{Stuck}(t) = \begin{cases} 1 & \text{if } \text{Var}(\mathbf{e}_{t-\tau:t}) < \epsilon \\ 0 & \text{otherwise} \end{cases} $$

$$ \gamma(t) = \gamma_{\text{base}} + \Delta \gamma \cdot \text{Stuck}(t) $$

**2. Nerves: Dynamic Masking of Local Joints**
Monitors each joint for anomalies ($p_{j,t}$). It dynamically redistributes weights, forcing dangerous joints to drop aggressive policies and switch to a safe policy $\delta_{i, \text{safe}}$ (relaxation or locking).

$$ w_{i,j}(\mathbf{e}_{t}, p_{j,t}) = \text{Softmax}(\mathbf{e}_{t})_{i} \times (1 - \text{Mask}(p_{j,t})) + \delta_{i, \text{safe}} \times \text{Mask}(p_{j,t}) $$

**3. Muscles: TinyMLP Ensemble**
Blends pre-trained ultra-lightweight policies $\pi_i$ with the nerve-adjusted weight vectors $\mathbf{w}_i$ using the Hadamard product (element-wise multiplication $\circ$) to output the final joint commands $\mathbf{u}_t$.

$$ \mathbf{u}_{t} = \sum_{i=1}^{N} \left( \mathbf{w}_{i}(\mathbf{e}_{t}, \mathbf{p}_{t}) \circ \pi_{i}(\mathbf{s}_{t}) \right) $$

#### Advantages (Pros)
* **Overwhelming Edge Performance & Stability:** Eliminates heavy matrix optimizations. With kinematic clipping (`np.clip`) and a hard limit on taboo arrays, it guarantees stable millisecond inference even on smartphone CPUs.
* **Instant Adaptation without Catastrophic Forgetting:** Fatal failures are added as absolute permanent repulsions. This instantly reflects safe behaviors—never repeating a mistake—without destroying the underlying walking policy.
* **Natural Emergence of Compensatory Movements:** Dynamic masking naturally generates biological behaviors like "limping to protect a hot joint" strictly through math, without complex if-else conditions.
* **Dynamic Bias for Environmental Adaptation:** Automatically distorts the potential field based on context (e.g., relaxing attraction gain when stuck or amplifying repulsion based on speed), ensuring escape from local optima and physical safety margins.

#### Limitations & Open Issues (Cons)
* **Ignoring ZMP & Fall Risk:** Sudden evasive or compensatory movements blend perfectly in math but cause rapid center-of-gravity shifts. If the lower-level hardware (PID/ZMP control) cannot absorb this, the policy shift may trigger a fall.
* **Dependence on Heuristic Parameters:** The system relies on manually tuned parameters. Incorrect initial values out of sync with physical traits can cause micro-vibrations (chattering).
* **Taboo Memory Consolidation:** While a hard limit prevents inference crashes, simply discarding old memories is suboptimal. Future updates require a mechanism (e.g., Gaussian Mixture Models) to spatially merge and compress old, adjacent memory points.

---

## License
This project is licensed under the MIT License - see the full text below.

**WITH GEMINI** - Co-authored / Generated with the assistance of AI (Gemini).

MIT License

Copyright (c) 2026 [Your Name/Organization]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.---

