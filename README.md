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
*
  ####計算例（わかりにくいとコメント頂いたので）
  気質ベクトル（\(\mathbf{e}_{t}\)）から丹田の球座標（\(R, \Theta, \Phi\)）への変換は、「2次元の心理・進行ベクトル」を「3次元の身体の構え」へ翻訳するプロセスです。
複雑な行列変換を使わず、大きさ（ノルム）と角度（atan2）の基本的な数式だけで、ロボットの「心理状態（気質）」を「人間らしい身体の所作」に直結させることができます。

1. 変数と基準値の定義

まず、変換の元になる「気質ベクトル」と、変換先の「丹田球座標」の基準値を設定します。
• 気質ベクトル \(\mathbf{e}_t = (e_x, e_y)\): ポテンシャル場から出力される2次元ベクトルです。ここでは「\(e_{x}\)：左右への回避・寄り添い意識」「\(e_{y}\)：前後への前進・警戒意識」とします。
• 丹田の基準球座標 \((R_0, \Theta_0, \Phi_0)\): ロボットがリラックスして直立している基本姿勢です。
• \(R_0 = 0.50\text{ m}\)（半径）： 丹田から手先・足先までの基本のリーチ（間合い）。
• \(\Theta_0 = 0^\circ\)（方位角）： 体の正面方向（水平角）。
• \(\Phi_0 = 0^\circ\)（仰角・タメ）： 丹田の高さの基準（プラスで背筋が伸び、マイナスで腰を落として構える）。

2. 変換の3つの数式ルール

気質ベクトル \(\mathbf{e}_{t}\) の「勢い（大きさ）」と「向き」から、球座標の各要素を以下の数式で決定します。

① 方位角 \(\Theta \)（体の向き・寄り添い角）

気質ベクトルが向いている方向へ、そのまま丹田の意識（体の向き）を向けます。
\(\Theta =\text{atan2}(e_{x},e_{y})\)
• 意味: 前進意識（\(e_{y}\)）が強ければ正面（\(0^{\circ }\)）を向き、横への回避や寄り添い（\(e_{x}\)）が強まると、自然とそちらへ体をひねります。

② 半径 \(R\)（間合い・リーチの伸縮）

気質ベクトルの大きさ（勢い） \(\Vert{}\mathbf{e}_t\Vert{} = \sqrt{e_x^2 + e_y^2}\) に応じて、手足の展開範囲 \(R\) を伸縮させます。
\(R=\text{clip}\left(R_{0}+k_{R}\cdot \|{}\mathbf{e}_{t}\|{},\ R_{\min },\ R_{\max }\right)\)
• 係数と制限: \(k_R = 0.15\)（伸び率）、最小半径 \(R_{\min} = 0.30\text{ m}\)（インナーゾーンの壁）、最大半径 \(R_{\max} = 0.70\text{ m}\)（腕・脚の限界長）とします。
• 意味: 勢いよく歩きたい時はリーチ（歩幅）が広くなり、迷って立ち止まると（\(\Vert{}\mathbf{e}_t\Vert{} \approx 0\)）、自然と手足を体に近い位置（\(0.50\text{ m}\)）へ収縮させます。

③ 仰角 \(\Phi \)（丹田のタメ・腰の落とし具合）

前後の意識 \(e_{y}\) と、左右の警戒・踏ん張り \(\vert{}e_x\vert{}\) から、重心の高さ（腰の落とし具合）を決めます。
\(\Phi =\text{clip}\left(k_{\text{fwd}}\cdot e_{y}-k_{\text{lat}}\cdot |{}e_{x}|{},\ -30^{\circ },\ +15^{\circ }\right)\)
• 係数: \(k_{\text{fwd}} = 10^\circ\)（前進時の伸び上がり）、\(k_{\text{lat}} = 20^\circ\)（横移動・回避時の腰の落とし込み）。
• 意味: まっすぐ歩く時は少し胸を張り（プラス角）、横に避けたり踏ん張ったりする時は重心を低くして安定させます（マイナス角）。

3. 具体的な数字を使ったシミュレーション（3つのシーン）

上記の数式に実際の数字を入れて、ロボットがどう動くか確認します。

シーンA：夜道をまっすぐ心地よくエスコート中

• 気質ベクトル \(\mathbf{e}_{t}\): \((0.0,\ 1.0)\) ※正面への意欲が高い状態
• 計算結果:
• ベクトルの大きさ: \(\sqrt{0.0^2 + 1.0^2} = 1.0\)
• 方位角 \(\Theta \): \(\text{atan2}(0.0, 1.0) = \mathbf{0^\circ}\)（まっすぐ正面）
• 半径 \(R\): \(0.50 + 0.15 \times 1.0 = \mathbf{0.65\text{ m}}\)（大股で堂々とした歩幅）
• 仰角 \(\Phi \): \(10^\circ \times 1.0 - 20^\circ \times 0.0 = \mathbf{+10^\circ}\)（少し背筋を伸ばした軽快な姿勢）

シーンB：右側から自転車が接近し、左へ「ビビリ回避」した瞬間

• 気質ベクトル \(\mathbf{e}_{t}\): \((-0.8,\ 0.2)\) ※永久タブーの斥力で左（マイナス）へ強く押された状態
• 計算結果:
• ベクトルの大きさ: \(\sqrt{(-0.8)^2 + 0.2^2} \approx 0.82\)
• 方位角 \(\Theta \): \(\text{atan2}(-0.8, 0.2) \approx \mathbf{-76^\circ}\)（左斜め方向へサッと体を向ける）
• 半径 \(R\): \(0.50 + 0.15 \times 0.82 \approx \mathbf{0.62\text{ m}}\)（回避のために足を踏み出す）
• 仰角 \(\Phi \): \(10^\circ \times 0.2 - 20^\circ \times 0.8 = 2^\circ - 16^\circ = \mathbf{-14^\circ}\)（腰をグッと落として踏ん張る防衛姿勢）

シーンC：障害物に囲まれて「スタック（迷い）」状態になった時

• 気質ベクトル \(\mathbf{e}_{t}\): \((0.05,\ -0.05)\) ※引力と斥力が釣り合い、動けなくなった状態
• 計算結果:
• ベクトルの大きさ: \(\sqrt{0.05^2 + (-0.05)^2} \approx 0.07\)
• 方位角 \(\Theta \): \(\text{atan2}(0.05, -0.05) = \mathbf{+135^\circ}\)（少し後ろを振り返るような迷いの向き）
• 半径 \(R\): \(0.50 + 0.15 \times 0.07 \approx \mathbf{0.51\text{ m}}\)（手足を体元に引き寄せた控えめな構え）
• 仰角 \(\Phi \): \(10^\circ \times (-0.05) - 20^\circ \times 0.05 = \mathbf{-1.5^\circ}\)（わずかに重心を落として立ち止まる）

4. この変換ルールの強み

この数式を通すことで、TinyMLPは「\(\mathbf{e}_{t}\) の微妙なニュアンスを汲み取って、係数（\(k_R, k_{\text{fwd}}, k_{\text{lat}}\)）を少し微調整するだけ」で済みます。
• すべてが連続的な三角関数と一次式で繋がっているため、「急に関節がガクッと動く」ことが絶対に起きません。
• さらに、clip 関数が上下限を常に監視しているため、どんなに強い斥力（タブー）が発生しても、ロボットは「腰を低く落として（\(\Phi = -30^\circ\)）、安全な間合い（\(R = 0.30\text{ m}\)）で身を守る」という最も人間らしい防御ポーズに自動で収束します。
### Example of calculation
## Transforming the Temperament Vector ($\mathbf{e}_t$) to Tanden Spherical Coordinates ($R, \Theta, \Phi$)

The transformation from the **Temperament Vector** ($\mathbf{e}_t$) to **Tanden Spherical Coordinates** ($R, \Theta, \Phi$) translates a "2D psychological/navigational vector" into a "3D physical posture."

Without relying on complex matrix transformations, this approach uses only basic norm (magnitude) and `atan2` (angle) formulas to directly map a robot's internal psychological state (temperament) to natural, human-like physical movements.

---

### 1. Definition of Variables and Baseline Values

First, define the source **Temperament Vector** and the baseline **Tanden Spherical Coordinates**.

* **Temperament Vector** $\mathbf{e}_t = (e_x, e_y)$: A 2D vector output from the potential field.
  * $e_x$: Lateral intent (avoidance / leaning in).
  * $e_y$: Longitudinal intent (forward drive / caution).
* **Baseline Tanden Spherical Coordinates** $(R_0, \Theta_0, \Phi_0)$: The robot's relaxed, upright neutral posture.
  * $R_0 = 0.50\text{ m}$ (**Radius**): Default reach/stride distance from the Tanden (center of gravity) to the end-effectors (hands/feet).
  * $\Theta_0 = 0^\circ$ (**Azimuth Angle**): Forward-facing horizontal orientation.
  * $\Phi_0 = 0^\circ$ (**Elevation Angle / Stance Depth**): Baseline Tanden height (positive = upright/extended; negative = lowered hips/braced stance).

---

### 2. Three Mathematical Rules for Transformation

Each spherical coordinate is computed from the magnitude (momentum) and direction of $\mathbf{e}_t$ as follows:

#### ① Azimuth Angle $\Theta$ (Body Orientation / Alignment)
Aligns the Tanden's orientation directly with the direction of the temperament vector:

$$
\Theta = \mathrm{atan2}(e_x, e_y)
$$

* **Meaning:** When forward intent ($e_y$) dominates, the robot faces straight ahead ($0^\circ$). As lateral avoidance or leaning ($e_x$) increases, the body naturally turns toward that direction.

#### ② Radius $R$ (Reach / Stride Extension)
Scales the limb deployment range $R$ according to the vector's magnitude (momentum) $\|\mathbf{e}_t\| = \sqrt{e_x^2 + e_y^2}$:

$$
R = \mathrm{clip}\left(R_0 + k_R \cdot \|\mathbf{e}_t\|,\ R_{\min},\ R_{\max}\right)
$$

* **Parameters & Limits:** Extension gain $k_R = 0.15$, minimum radius $R_{\min} = 0.30\text{ m}$ (inner-zone boundary), maximum radius $R_{\max} = 0.70\text{ m}$ (physical limb limit).
* **Meaning:** High momentum produces a wider stride/reach, while hesitation or stopping ($\|\mathbf{e}_t\| \approx 0$) naturally draws the limbs back toward the body ($0.50\text{ m}$).

#### ③ Elevation Angle $\Phi$ (Stance Depth / Center of Gravity)
Determines hip height (how low the robot crouches) from longitudinal intent ($e_y$) and lateral caution/bracing ($|e_x|$):

$$
\Phi = \mathrm{clip}\left(k_{\text{fwd}} \cdot e_y - k_{\text{lat}} \cdot |e_x|,\ -30^\circ,\ +15^\circ\right)
$$

* **Parameters:** Forward lift gain $k_{\text{fwd}} = 10^\circ$ (straightening up while walking forward), lateral drop gain $k_{\text{lat}} = 20^\circ$ (lowering the hips during lateral movement or avoidance).
* **Meaning:** Walking straight lifts the posture slightly (positive angle), whereas dodging sideways or bracing lowers the center of gravity for stability (negative angle).

---

### 3. Numerical Simulations (3 Scenarios)

#### Scenario A: Escorting smoothly straight down a night street
* **Temperament Vector $\mathbf{e}_t$:** $(0.0,\ 1.0)$ *(strong forward intent)*
* **Results:**
  * **Magnitude:** $\sqrt{0.0^2 + 1.0^2} = 1.0$
  * **Azimuth $\Theta$:** $\mathrm{atan2}(0.0, 1.0) = \mathbf{0^\circ}$ *(facing straight ahead)*
  * **Radius $R$:** $0.50 + 0.15 \times 1.0 = \mathbf{0.65\text{ m}}$ *(confident, wide stride)*
  * **Elevation $\Phi$:** $10^\circ \times 1.0 - 20^\circ \times 0.0 = \mathbf{+10^\circ}$ *(lightly upright, breezy posture)*

#### Scenario B: Startled dodge to the left as a bicycle approaches from the right
* **Temperament Vector $\mathbf{e}_t$:** $(-0.8,\ 0.2)$ *(strongly pushed left by a permanent-taboo repulsive force)*
* **Results:**
  * **Magnitude:** $\sqrt{(-0.8)^2 + 0.2^2} \approx 0.82$
  * **Azimuth $\Theta$:** $\mathrm{atan2}(-0.8, 0.2) \approx \mathbf{-76^\circ}$ *(quickly turns body diagonally left)*
  * **Radius $R$:** $0.50 + 0.15 \times 0.82 \approx \mathbf{0.62\text{ m}}$ *(steps out to evade)*
  * **Elevation $\Phi$:** $10^\circ \times 0.2 - 20^\circ \times 0.8 = 2^\circ - 16^\circ = \mathbf{-14^\circ}$ *(drops hips low into a braced defensive stance)*

#### Scenario C: Surrounded by obstacles and "stuck" (hesitation state)
* **Temperament Vector $\mathbf{e}_t$:** $(0.05,\ -0.05)$ *(attractive and repulsive forces cancel out)*
* **Results:**
  * **Magnitude:** $\sqrt{0.05^2 + (-0.05)^2} \approx 0.07$
  * **Azimuth $\Theta$:** $\mathrm{atan2}(0.05, -0.05) = \mathbf{+135^\circ}$ *(hesitant angle, glancing slightly back)*
  * **Radius $R$:** $0.50 + 0.15 \times 0.07 \approx \mathbf{0.51\text{ m}}$ *(compact stance with limbs drawn close)*
  * **Elevation $\Phi$:** $10^\circ \times (-0.05) - 20^\circ \times 0.05 = \mathbf{-1.5^\circ}$ *(halts while slightly lowering center of gravity)*

---

### 4. Strengths of This Transformation Rule

By routing control through these equations, a `TinyMLP` only needs to capture subtle nuances in $\mathbf{e}_t$ and slightly fine-tune the coefficients ($k_R, k_{\text{fwd}}, k_{\text{lat}}$).

* **Smoothness:** Because everything is linked via continuous trigonometric and linear functions, sudden jerky joint movements are eliminated.
* **Built-in Safety:** Because the `clip` function continuously enforces upper and lower bounds, even under extreme repulsive forces (taboos), the robot automatically converges to a natural, human-like defensive posture—lowering its hips ($\Phi = -30^\circ$) and maintaining a safe personal boundary ($R = 0.30\text{ m}$).
# 📝Another Concept Note: Tanden-Centric Motion Primitives (丹田原点・球座標による運動プリミティブ設計論)

## 1. Core Concept (概要)
In human and robot body control, we abandon the brutal-force approach of heavily computing world coordinates and complex inverse kinematics (IK). Instead, we establish **the "Tanden" (Center of Mass/Gravity Core) as the sole origin (spherical coordinate system)**, subordinating terminal links as geometric constraints (adjustment of discrepancies). This architecture achieves ultra-lightweight and smooth motion.

人間やロボットの身体制御において、世界座標や複雑な逆運動学（IK）の計算をゴリゴリ解くアプローチを捨て、**「丹田（重心）」を唯一の原点（球座標系）**に据え、末端のリンクを幾何学的な制約（辻褄合わせ）として従属させることで、超軽量かつ滑らかな動作を実現するアーキテクチャ。

---

## 2. Architectural Pillars (三大原則)

1. **The Command Tower is "Tanden Spherical Coordinates" and "Movement Intent"**
   - Spatial awareness is entirely handled via polar coordinates $(R, \Theta, \Phi)$ centered on the Tanden.
   - The source of movement is solely abstract parameters: global "movement speed" and "stride/reach" (expansion range on the spherical surface).
   - **司令塔は「丹田の球座標」と「移動の意図」**
     - 空間の把握はすべて丹田を中心とした極座標 $(R, \Theta, \Phi)$ で行う。
     - 動きの源泉は、グローバルな「移動速度」と「歩幅（球面上の展開範囲）」の抽象パラメータのみ。
2. **Terminal Links are "Merely Geometric Constraints"**
   - Joints like shoulders, elbows, and wrists are just mathematical adjustments (such as the law of cosines) to match oneself to the target position.
   - Instead of forcing AI or microcontrollers to compute heavy IK, formulas handle the heavy lifting.
   - **末端のリンクは「ただの幾何学的制約」**
     - 肩・肘・手首などの関節は、ターゲットの位置に自分を合わせるための数式（余弦定理など）の辻褄合わせにすぎない。
     - 複雑なIKをAIやマイコンに全計算させず、数式に肩代わりさせる。
3. **Muddy Physics is Delegated to "Hardware Feedback"**
   - Without self-simulating gravity, inertia, or ground reaction force (GRF), we incorporate feedback from hardware sensors (power, acceleration, load) as proprioception.
   - **泥臭い物理は「実機のフィードバック」に委ねる**
     - 重力、慣性、地面からの反力（GRF）を自前でシミュレートせず、実機センサー（電力・加速度・負荷）からのフィードバックを体性感覚として取り込む。

---

## 3. Math & Logic (計算モデルの骨子)

- **Target and Link Interlocking (ターゲットとリンクの連動):**
  - Target end-effector position from Tanden origin / 丹田原点の手先目標：$\mathbf{P} = (R \cos\Theta, R \sin\Theta)$
  - For linear distance $D$ derived from the offset between Tanden and shoulder, the elbow angle $\beta$ is uniquely determined by the law of cosines:
    $$\cos(\beta) = \frac{L_1^2 + D^2 - L_2^2}{2 L_1 D}$$
- **Lightweight Self-Collision Check (軽量な自己衝突チェック):**
  - Without heavy polygon collision checks, collisions are instantaneously prevented using **"radius $r$ thresholds (inner zones)"** and **"azimuth $\Theta$ exclusion masks (inequalities)"**.
  - 重いポリゴン判定はせず、**「半径 $r$ の閾値（インナーゾーン）」**や**「方位角 $\Theta$ の排他マスク（不等式）」**だけで瞬時に弾く。

---

## 4. Division of Labor with "MLP (AI)" (「MLP（AI）」との分業体制)
- **AI (MLP) Role / AI（MLP）の仕事:** 
  Handles exclusively high-level decision-making (primitive selection)—such as how to alter Tanden spherical coordinate parameters, stride, and speed based on the current context ("pouring a drink," "leaning in," "supporting"). Since input dimensions are extremely small, the model becomes dramatically lightweight.
  現在の文脈（「お酌をする」「身を寄せる」「支える」）に応じて、丹田の球座標パラメータや歩幅・速度をどう変化させるべきかという**高次な意思決定（プリミティブの選択）**のみを担当する。入力次元が極めて小さいため、モデルが劇的に軽くなる。
- **Geometry (Formula) Role / 幾何学（数式）の仕事:** 
  Instantly resolves link and collision-avoidance constraints toward the selected target position.
  選ばれたターゲット位置に向かって、リンクや衝突回避の制約を瞬時に解決する。

---

## 5. Application Examples (アプリケーションの展開例)

- **Dictionary of Everyday Etiquette (日常の作法の辞書化):**
  - Human-like "forms" and "pauses (ma)" can be cleanly coded using combinations of a few parameters, such as Tanden hesitation (height adjustment) and radius $r$ contraction.
  - 丹田のタメ（高さの調整）や半径 $r$ の収縮という数個のパラメータの組み合わせで、人間らしい「型」や「間（ま）」が綺麗にコード化できる。

---
> **Summary (総括):**
> While unsuited for physical-limit acrobatics like circus tricks, this is a new paradigm for realizing **"everyday etiquette, walking, and safe mutual support"** tailored to human society using edgy, lightweight, and elegant code.
> サーカスのような「物理限界の曲芸」には向かないが、人間社会に寄り添う**「日常の作法、歩行、安全な支え合い」**をエッジの効いた軽量かつ優美なコードで実現するための、パラダイムである。


```python
==============================================================================
Emotion-Tabu Edge Biped: 脳・神経・筋肉 完全統合アーキテクチャ
(対話・歩幅学習・ビビリ防衛・安全時詳細識別 網羅版) + 動的バイアス実装
==============================================================================
"""

import numpy as np
import time
import socket
import json
import threading
from collections import deque

def get_rotation_matrix(theta_deg):
    theta_rad = np.radians(theta_deg)
    c, s = np.cos(theta_rad), np.sin(theta_rad)
    return np.array(((c, -s), (s, c)))

def softmax(x):
    e_x = np.exp(x - np.max(x))
    return e_x / e_x.sum(axis=0)

class TinyMLPPolicy:
    """筋肉: 事前学習済みの超軽量歩行ポリシー"""
    def __init__(self, num_joints):
        self.num_joints = num_joints
    
    def predict(self, state):
        return np.random.uniform(-1.0, 1.0, self.num_joints)

class DualBrainRobot:
    def __init__(self):
        # --- 1. Brain (気質とナビゲーション状態) ---
        self.e_t = np.array([0.0, 0.0])
        self.waypoints = deque()
        self.history = deque(maxlen=40)
        
        self.soft_taboos = []
        self.fatal_taboos = []
        
        # [工夫: 永久タブーのハードリミット設定]
        # 理由: 稼働時間が延びた際、計算するべきガウス関数が無限に増えて
        # エッジ推論（ミリ秒）が破綻するのを防ぐための安全装置。
        self.MAX_FATAL_TABOOS = 50 
        
        # --- 身体モデル (固有受容覚と物理制約) ---
        self.SHOULDER_WIDTH = 0.60  # 肩幅 (m) : 迂回時のゆとり
        self.current_stride = 0.20  # 初期推定歩幅 (m)
        self.current_speed = 0.0    # 推定速度 (m/s)
        self.last_valid_dist = -1.0
        self.last_calib_time = time.time()
        self.accZ_peaks = 0         
        self.current_accZ = 1.0     
        
        # --- 状態フラグとコンテキスト ---
        self.is_active_task = False
        self.is_stuck = False
        self.wait_for_direction = False
        self.hardware_error = False
        self.visible_landmarks = {"横断歩道": np.array([0.0, 3.0]), "ビル": np.array([-2.0, 5.0])}
        
        # --- 【新規追加】安全時の詳細識別（システム2）に関する状態 ---
        self.latest_object_label = "none" # 重いモデルによる識別結果
        self.object_confidence = 0.0
        
        # --- 2. Nerves (局所神経の状態) ---
        self.NUM_JOINTS = 6 # 両足6自由度
        self.p_t = np.zeros(self.NUM_JOINTS) # 関節の危険度・温度 (0.0~1.0)
        self.delta_safe = np.array([0.0, 0.0, 1.0]) # 安全ポリシー(配列末尾)への強制重み
        
        # --- 3. Muscles (TinyMLPアンサンブル) ---
        self.policies = [TinyMLPPolicy(self.NUM_JOINTS) for _ in range(3)]
        
        self.lock = threading.Lock()
        self.sock_send = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    def send_interrupt(self, text):
        """TALK.pyの雑談を強制キルし、タスク脳の言葉を口から発する"""
        payload = {"cmd": "interrupt", "text": text}
        self.sock_send.sendto(json.dumps(payload, ensure_ascii=False).encode('utf-8'), ("127.0.0.1", 5002))
        print(f"\n[タスク脳 👄 割込発話] 「{text}」\n")

    def get_current_heading(self):
        """直近の移動履歴から、ロボットの現在の進行方向ベクトルを取得"""
        if len(self.history) < 2: return np.array([0.0, 1.0])
        vec = self.history[-1] - self.history[0] 
        norm = np.linalg.norm(vec)
        return (vec / norm) if norm > 0.001 else np.array([0.0, 1.0])

    # ------------------------------------------------------------------------
    # [スレッド1] STTコマンドの並行受信 (経路指示・緊急停止・訂正の受容)
    # ------------------------------------------------------------------------
    def listen_stt(self):
        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.bind(("0.0.0.0", 5010))
        while True:
            data, _ = sock.recvfrom(4096)
            text = json.loads(data.decode('utf-8')).get("text", "")
            if not text: continue

            with self.lock:
                if any(w in text for w in ["止まれ", "ストップ", "危ない"]):
                    heading = self.get_current_heading()
                    # [工夫: 上限到達時は最も古いタブーを押し出す]
                    if len(self.fatal_taboos) >= self.MAX_FATAL_TABOOS:
                        self.fatal_taboos.pop(0)
                    self.fatal_taboos.append(self.e_t.copy() + heading * 0.5)
                    self.waypoints.clear()
                    self.send_interrupt("危ない！止まります！")
                    continue

                if any(w in text for w in ["帰ろう", "家", "帰る"]):
                    self.is_active_task = True
                    self.hardware_error = False
                    self.waypoints.clear()
                    self.waypoints.append(np.array([10.0, 10.0]))
                    self.send_interrupt("はい！お家に帰りましょう。ついてきてね。")
                    continue

                if self.is_active_task and self.wait_for_direction:
                    heading = self.get_current_heading()
                    if "違う" in text or "ダメ" in text:
                        self.soft_taboos.append({"pos": self.e_t.copy() + heading * 1.0, "time": time.time()})
                        self.send_interrupt("あっ、ごめんなさい。こっちじゃないのね。")
                        continue

                    for lm, rel_pos in self.visible_landmarks.items():
                        if lm in text:
                            self.waypoints.append(self.e_t + rel_pos)
                            self.is_stuck, self.wait_for_direction = False, False
                            self.send_interrupt(f"{lm}ね、わかったわ！")
                            break
                    else:
                        if "右" in text:
                            target = self.e_t + get_rotation_matrix(-90).dot(heading) * 3.0
                            self.waypoints.append(target)
                            self.is_stuck, self.wait_for_direction = False, False
                            self.send_interrupt("右ね、ありがとう！")
                        elif "左" in text:
                            target = self.e_t + get_rotation_matrix(90).dot(heading) * 3.0
                            self.waypoints.append(target)
                            self.is_stuck, self.wait_for_direction = False, False
                            self.send_interrupt("左ね、わかったわ。")

    # ------------------------------------------------------------------------
    # [スレッド2] 視覚センサー (マクロな一発退場)
    # ------------------------------------------------------------------------
    def listen_vision(self):
        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.bind(("0.0.0.0", 5007))
        while True:
            data, _ = sock.recvfrom(1024)
            payload = json.loads(data.decode('utf-8'))
            status = payload.get("status", "")
            
            with self.lock:
                if status == "danger_large_object":
                    heading = self.get_current_heading()
                    # [工夫: 上限到達時は最も古いタブーを押し出す]
                    if len(self.fatal_taboos) >= self.MAX_FATAL_TABOOS:
                        self.fatal_taboos.pop(0)
                    self.fatal_taboos.append(self.e_t.copy() + heading * 1.0)
                    self.waypoints.clear()
                    self.send_interrupt("危ない！大きなものが来ます！")
                
                # 【新規追加】安全時における重い識別モデルからの結果受給ポート
                elif status == "object_identified":
                    self.latest_object_label = payload.get("label", "unknown")
                    self.object_confidence = payload.get("confidence", 0.0)

    # ------------------------------------------------------------------------
    # [スレッド3] 足元LiDAR・IMU (ミクロな精密回避と歩幅学習)
    # ------------------------------------------------------------------------
    def listen_lidar(self):
        sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        sock.bind(("0.0.0.0", 5006))
        while True:
            data, _ = sock.recvfrom(1024)
            payload = json.loads(data.decode('utf-8'))
            status = payload.get("status", "")
            dist_m = payload.get("dist", -1) / 100.0
            self.current_accZ = payload.get("accZ", 1.0)
            
            if abs(self.current_accZ - 1.0) > 0.3:
                self.accZ_peaks += 1

            current_time = time.time()
            with self.lock:
                if status == "error" and not self.hardware_error:
                    self.hardware_error = True
                    self.send_interrupt("足元のセンサーが応答しないわ。安全のために止まるね。")
                    continue

                if status == "ok":
                    heading = self.get_current_heading()
                    
                    if dist_m < 0.8:
                        self.last_valid_dist = -1.0 
                        if dist_m < 0.3:
                            # [工夫: 上限到達時は最も古いタブーを押し出す]
                            if len(self.fatal_taboos) >= self.MAX_FATAL_TABOOS:
                                self.fatal_taboos.pop(0)
                            self.fatal_taboos.append(self.e_t.copy() + heading * dist_m)
                            self.send_interrupt("危ない！")
                        else:
                            self.soft_taboos.append({"pos": self.e_t.copy() + heading * dist_m, "time": current_time})
                        continue

                    if self.last_valid_dist > 0 and self.is_active_task:
                        dt = current_time - self.last_calib_time
                        if dt > 1.0:
                            delta_dist = self.last_valid_dist - dist_m
                            if 0.05 < delta_dist < 1.0:
                                self.current_speed = delta_dist / dt
                                if self.accZ_peaks > 0:
                                    new_stride = delta_dist / self.accZ_peaks
                                    self.current_stride = (self.current_stride * 0.7) + (new_stride * 0.3)
                    
                    self.last_valid_dist = dist_m
                    if current_time - self.last_calib_time > 1.0:
                        self.last_calib_time = current_time
                        self.accZ_peaks = 0

    # ------------------------------------------------------------------------
    # [メインループ] 脳(ポテンシャル) -> 神経(マスキング) -> 筋肉(アダマール積)
    # ------------------------------------------------------------------------
    def update_control_loop(self):
        while True:
            time.sleep(0.05)
            with self.lock:
                # 【条件厳守】タスク中であり、かつセンサー異常や加速度異常がない安全な時のみ動作
                is_safe_condition = (
                    self.is_active_task and 
                    not self.hardware_error and 
                    abs(self.current_accZ - 1.0) <= 0.3
                )
                if not is_safe_condition: 
                    continue

                current_time = time.time()
                heading = self.get_current_heading()

                # ========================================================
                # 1. 【脳】 Dual Tabu Potential Field (引力と斥力の合算)
                # ========================================================
                grad_goal = np.zeros(2)
                if self.waypoints:
                    diff_goal = self.waypoints[0] - self.e_t
                    if np.linalg.norm(diff_goal) < 0.5:
                        self.waypoints.popleft()
                        if not self.waypoints:
                            self.send_interrupt("ふふ、無事に着いたわ。一緒に帰ってくれてありがとう！")
                            self.is_active_task = False
                            continue
                    else:
                        # [工夫: スタック時の引力ゲイン緩和]
                        # 理由: 局所解（スタック）に陥っている時は、目的地へ向かおうとする力が
                        # 脱出の妨げになるため、引力を下げて迂回・反発（斥力）を優先させる。
                        dynamic_goal_gain = 0.1 if self.is_stuck else 0.5
                        grad_goal = dynamic_goal_gain * diff_goal

                grad_soft = np.zeros(2)
                gamma = 0.5 if self.is_stuck else 0.05
                active_soft = []
                for tabu in self.soft_taboos:
                    dt = current_time - tabu["time"]
                    decay = np.exp(-gamma * dt)
                    if decay > 0.01:
                        diff = self.e_t - tabu["pos"]
                        dist = np.linalg.norm(diff) + 1e-5
                        push_force = (diff / dist) / (dist**2)
                        
                        # ====================================================
                        # 【身体モデル統合】歩幅・速度・肩幅を反映したスムーズな迂回
                        # ====================================================
                        # 自身の「現在の肩幅(SHOULDER_WIDTH)」に、
                        # 「現在の速度(current_speed)」および「学習した歩幅(current_stride)」の係数をかけ合わせ、
                        # 安全かつ優雅に避けるための動的なスリップ力を生成する。
                        speed_factor = np.clip(self.current_speed / 1.0, 0.5, 1.5)
                        stride_factor = self.current_stride / 0.20
                        dynamic_width = self.SHOULDER_WIDTH * speed_factor * stride_factor
                        
                        # 【新規追加】安全時に重いモデルで識別したオブジェクトの種類に応じた補正
                        if self.latest_object_label == "person":
                            # 人間の場合は少し大きめに余裕をもって避ける
                            dynamic_width *= 1.3
                        elif self.latest_object_label == "static_obstacle":
                            # 静止物の場合は必要最小限の肩幅でスマートに交わす
                            dynamic_width *= 1.0

                        lateral_vec = get_rotation_matrix(90).dot(heading)
                        slip_force = lateral_vec * (dynamic_width / dist)
                        
                        grad_soft -= decay * (push_force + slip_force)
                        active_soft.append(tabu)
                self.soft_taboos = active_soft

                grad_fatal = np.zeros(2)
                
                # [工夫: 速度に応じた斥力係数のスケーリング]
                # 理由: ロボットの移動速度が速い時は、慣性による制動距離やハードウェアの
                # 重心吸収マージンを考慮し、より遠くから強く反発させるためゲインを増幅する。
                speed_ratio = np.clip(self.current_speed / 0.5, 1.0, 2.0)
                dynamic_fatal_gain = 5.0 * speed_ratio
                
                for f_pos in self.fatal_taboos:
                    diff = self.e_t - f_pos
                    grad_fatal -= dynamic_fatal_gain * (diff / np.linalg.norm(diff)**3)

                # 合力と歩幅制限(Kinematic Clipping)による気質ベクトル更新
                total_force = grad_goal + grad_soft + grad_fatal
                max_step = self.current_stride * 0.2
                self.e_t += np.clip(0.02 * total_force, -max_step, max_step)
                self.history.append(self.e_t.copy())

                # メタ・タブーサーチ (スタック検知とヘルプ要請)
                if len(self.history) == self.history.maxlen and not self.wait_for_direction:
                    variance = np.var(list(self.history), axis=0).sum()
                    if variance < 0.005 and abs(self.current_accZ - 1.0) < 0.1 and not self.is_stuck:
                        self.is_stuck, self.wait_for_direction = True, True
                        lm = list(self.visible_landmarks.keys())[0] if self.visible_landmarks else None
                        msg = f"ごめんなさい、道が分からなくなっちゃった。あの{lm}の方へ行くんだっけ？" if lm else "ごめんなさい、どっちに行けばいいか教えて？"
                        self.send_interrupt(msg)

                # ========================================================
                # 2. 【局所神経】 動的マスキング
                # ========================================================
                danger_mask = np.where(self.p_t > 0.8, 1.0, 0.0)
                base_weights = softmax(np.append(self.e_t, 0.0)) 

                # ========================================================
                # 3. 【筋肉】 TinyMLPの実行と代償動作の生成
                # ========================================================
                s_t = np.random.uniform(0, 1, 10) 
                u_t = np.zeros(self.NUM_JOINTS)
                
                for i, policy in enumerate(self.policies):
                    w_i = base_weights[i] * (1 - danger_mask) + self.delta_safe[i] * danger_mask
                    u_t += w_i * policy.predict(s_t)
                
                # send_to_servos(u_t) # モーターへ送信

def main():
    robot = DualBrainRobot()
    threading.Thread(target=robot.listen_stt, daemon=True).start()
    threading.Thread(target=robot.listen_vision, daemon=True).start()
    threading.Thread(target=robot.listen_lidar, daemon=True).start()
    robot.update_control_loop()

if __name__ == "__main__":
    main()



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

