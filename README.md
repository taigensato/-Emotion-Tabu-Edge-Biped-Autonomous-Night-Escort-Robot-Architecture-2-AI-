markdown
# Emotion-Tabu Edge Biped: Autonomous Night-Escort Robot Architecture
**感情タブーサーチによる超軽量2足歩行エッジAI：深夜の酔っ払い帰宅エスコート・ロボット構想**

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
本プロジェクトは、私が以前に開発・公開した**「日本人女性バーテンダーとのオフライン会話ドロイド」の発想を、身体（2足歩行制御）へと直接拡張しようという構想**です。前作の対話システムでは、軽量LLM（`Gemma 2B`）に「会話ドメインの判定」のみを担当させ、実際の応答生成を「Eliza的辞書 ＋ タブーサーチ（同じ返答や不自然な返答の回避）」で行うことで、クラウドに頼らず人間らしい間合いと多様性を実現しました。

**核心となるアイデア：** *「会話で成功した『タブーサーチ × 超軽量モデル』の仕組みを、そのまま2足歩行の歩き方（歩容）制御に拡張できないか？」*
バーテンダーの対話脳と軽量の身体制御エンジンを連動させることで、**深夜の暗い夜道で酔っ払いを家まで送り届ける自律ロボット**を目指します。通りすがりの人に道を尋ね、相手の表情（「それ変ダヨ」という困惑顔）や路面の状況を読み取りながら、リアルタイムに歩き方や態度を変化させる仕組みをこれから実装していきます。

---

## 2. Key Architectural Innovations / 技術的特徴（設計構想）

This project is a challenge as a hobbyist robot developer to create an autonomous bipedal robot with environmental adaptation and emotional expression using only millisecond inference on an Android device (Python + NumPy on Termux), without relying on expensive GPUs or massive Deep Reinforcement Learning (DRL) models. 
It implements a 3-layer structure ("Brain for overall policy", "Nerves for local anomaly management", and "Muscles for execution") into a lightweight mathematical model, eliminating heavy matrix computations.

本プロジェクトは、高価なGPUや巨大な深層強化学習（DRL）モデルに頼らず、Android端末（Termux上のPython + NumPy）のみのミリ秒推論で、環境適応と感情表現を持つ2足歩行ロボットを作るというホビーロボット開発者としての挑戦です。
「全体の方針を決める脳」「局所的な異常を管理する神経」「運動を実行する筋肉」の3層構造を、重い行列計算を排した軽量な数式モデルに落とし込んでいます。
---

## 🧠 1. Brain: Temperament Vector Update (Dual Tabu Potential Field)
**【脳】気質ベクトルの更新（Dual Tabu Potential Field）**

The overall "emotion/temperament vector $\mathbf{e}_t$" is updated every step by the resultant force of attraction to a goal and repulsion from past failure memories (two types of taboos).
ロボット全体の「感情・気質ベクトル $\mathbf{e}_t$」は、目標への引力と、過去の失敗記憶からの斥力（2種類のタブー）の合力によって毎ステップ更新されます。

$$ \mathbf{e}_{t+1} = \mathbf{e}_{t} - \alpha \nabla_{\mathbf{e}} \left( U_{\text{goal}}(\mathbf{e}_{t}) + U_{\text{human}}(\mathbf{e}_{t}) + U_{\text{fatal}}(\mathbf{e}_{t}) \right) $$

**① Attraction to the goal (Quadratic potential) / 目標への引力（2次ポテンシャル）**
A spring-like force pulling back to the ideal temperament $\mathbf{e}_{\text{target}}$.
理想的な気質 $\mathbf{e}_{\text{target}}$ へ引き戻すバネの力。
$$ U_{\text{goal}}(\mathbf{e}_{t}) = \frac{1}{2} k_{\text{goal}} \|\mathbf{e}_{t} - \mathbf{e}_{\text{target}}\|^2 $$

**② Repulsion from Soft Taboo (Time-decaying Gaussian) / 一時タブーからの斥力（時間減衰ガウス関数）**
Force to keep away from "soft failure memories $\mathcal{H}_{\text{human}}$" like being scolded by humans. Decays over time.
人間に怒られた等の「ソフトな失敗記憶 $\mathcal{H}_{\text{human}}$」から遠ざける力。時間経過で忘却します。
$$ U_{\text{human}}(\mathbf{e}_{t}) = \sum_{j \in \mathcal{H}_{\text{human}}} A_{\text{soft}} e^{-\gamma (t - t_j)} \exp\left( -\frac{\|\mathbf{e}_{t} - \mathbf{e}_j\|^2}{2\sigma_{\text{soft}}^2} \right) $$

**③ Repulsion from Fatal Taboo (Permanent Gaussian) / 永久タブーからの斥力（永続ガウス関数）**
Absolute barrier against "fatal failure memories $\mathcal{H}_{\text{fatal}}$" like falling over. Does not decay.
転倒などの「致命的な失敗記憶 $\mathcal{H}_{\text{fatal}}$」に対する絶対障壁。減衰しません。
$$ U_{\text{fatal}}(\mathbf{e}_{t}) = \sum_{k \in \mathcal{H}_{\text{fatal}}} A_{\text{fatal}} \exp\left( -\frac{\|\mathbf{e}_{t} - \mathbf{e}_k\|^2}{2\sigma_{\text{fatal}}^2} \right) $$

---

## 🧬 2. Brain Plasticity: Dynamic Adaptation via Meta-Tabu Search
**【脳の可塑性】メタ・タブーサーチによる動的適応**

Detects "local optima (stuck state)" surrounded by taboo repulsions and autonomously rewrites parameters to escape.
タブーの斥力に囲まれて身動きが取れない「局所解（スタック）」を検知し、自律的にパラメータを書き換えて脱出します。

**Stuck detection / スタック検知:** 
Determines if the variance of temperament movement falls below a threshold $\epsilon$.
気質の移動量（分散）が閾値 $\epsilon$ を下回ったか判定。
$$ \text{Stuck}(t) = \begin{cases} 1 & \text{if } \text{Var}(\mathbf{e}_{t-\tau:t}) < \epsilon \\ 0 & \text{otherwise} \end{cases} $$

**Dynamic relaxation example / 動的緩和の例:** 
When stuck, it accelerates the forgetting of soft taboos.
スタック時は、一時タブーの忘却を加速させます。
$$ \gamma(t) = \gamma_{\text{base}} + \Delta \gamma \cdot \text{Stuck}(t) $$

---

## ⚡ 3. Local Nerves: Joint-Space Potential and Dynamic Masking
**【局所神経】関節空間ポテンシャルと動的マスキング**

Instead of blindly executing the overall temperament command, it evaluates the state of each joint (overheating, failure, etc.) and dynamically filters the policy weights for each joint.
全体の気質命令をそのまま実行するのではなく、関節ごとの状態（過熱、故障など）を評価し、関節ごとにポリシーの重み付けを動的にフィルタリングします。

**Local potential $\mathbf{p}_t$ and weight redistribution / 局所ポテンシャル $\mathbf{p}_t$ と重みの再配分:**
When the danger level of joint $j$ increases, the weight of aggressive policies is forced to zero, applying the weight of a safe policy (relaxation or locking) $\delta_{i, \text{safe}}$.
関節 $j$ の危険度（温度やエラー値）が高まった場合、アグレッシブなポリシーの重みを強制的にゼロにし、安全なポリシー（脱力や固定）の重み $\delta_{i, \text{safe}}$ を適用します。

$$ w_{i,j}(\mathbf{e}_{t}, p_{j,t}) = \text{Softmax}(\mathbf{e}_{t})_{i} \times (1 - \text{Mask}(p_{j,t})) + \delta_{i, \text{safe}} \times \text{Mask}(p_{j,t}) $$

This naturally manifests compensatory movements (e.g., "limping on the right leg because the right knee is hot") without post-learning.
これにより、「右膝が熱いので右足だけかばって歩く（代償動作）」が事後学習なしで自然発現します。

---

## 🦾 4. Muscles: Execution of TinyMLP Ensemble
**【筋肉】TinyMLPアンサンブルの実行**

The final joint command $\mathbf{u}_t$ is output by blending pre-trained ultra-lightweight policies $\pi_i$ with the "weight vector for each joint $\mathbf{w}_i$" using element-wise multiplication (Hadamard product $\circ$).
最終的な関節指令 $\mathbf{u}_t$ は、事前学習済みの超軽量ポリシー $\pi_i$ に対し、「関節ごとの重みベクトル $\mathbf{w}_i$」を要素ごとの掛け算（アダマール積 $\circ$）でブレンドして出力します。

$$ \mathbf{u}_{t} = \sum_{i=1}^{N} \left( \mathbf{w}_{i}(\mathbf{e}_{t}, \mathbf{p}_{t}) \circ \pi_{i}(\mathbf{s}_{t}) \right) $$

---

## 💡 Pros / プロジェクトの強み
* **Overwhelming Edge Performance / 圧倒的なエッジ動作:** No expensive GPUs/AI chips needed. Math-based potential fields and simple matrix ops enable millisecond autonomous control on smartphone CPUs. (高価なGPUやAIチップは不要。数式ベースのポテンシャル場と単純な行列演算により、スマホのCPU環境でもミリ秒単位の自律制御が可能です。)
* **Instant Learning without Catastrophic Forgetting / 破滅的忘却のない即時学習:** Memories like "falling" or "scolded" are added as potential repulsions, instantly reflecting safe behaviors without breaking original walking abilities. (「転んだ」「怒られた」という記憶をポテンシャルの斥力として後乗せするため、元の歩行能力を壊すことなく、1秒前の失敗を二度と繰り返さない安全行動を即時反映できます。)
* **Natural Emergence of Compensatory Movements / 代償動作の自然発現:** Joint-by-joint potential control allows switching only overheated/failed parts to safe policies, realizing "limping/protecting pain" without branch programming. (関節ごとのポテンシャル制御により、故障や過熱を検知した部位だけを安全ポリシーに切り替える「痛みをかばう歩き方」がプログラムの分岐なしで実現します。)

## ⚠️ Open Issues & Cons / 理論の穴・今後の課題
This model is a conceptual personal project, and the following unresolved issues (weaknesses) exist for real-world deployment. (本モデルは個人開発のコンセプトであり、実機投入には以下の未解決課題が存在します。)

1. **Ignoring ZMP (Zero Moment Point) & Fall Risk / ZMPの無視と転倒リスク**
   Sudden compensatory movements blend perfectly in math but cause rapid center-of-gravity shifts in physics. If the robot's high-frequency PID control cannot absorb this, the policy change itself may cause a fall. (関節ポテンシャルで急にかばう動作をした際、数式上は綺麗にブレンドされますが、物理世界では急激な重心移動が発生します。ロボット側のPID制御で吸収しきれない場合、転倒する危険があります。)
2. **Hyperparameter Tuning Swamp / パラメータチューニングの沼**
   Many heuristic parameters exist (e.g., attraction gain). Though meta-tabu search self-relaxes them, wrong initial values can cause the robot to micro-vibrate (chatter). (ポテンシャルの引力ゲインなど、ヒューリスティックに設定すべきパラメータが多く存在します。初期値の設定を誤るとロボットが微振動を起こし使い物になりません。)
3. **Taboo Memory Bloat (Curse of Dimensionality) / タブーメモリの肥大化**
   Accumulating permanent taboos infinitely increases the Gaussian functions to compute, eventually breaking the inference speed limit. Logic to merge/compress old memories (e.g., Gaussian Mixture Models) is needed. (永久タブーが蓄積しすぎると、計算するべきガウス関数が無限に増え、やがて推論速度の制約を破綻させます。古い記憶を統合・圧縮するロジックが将来的に必要です。)
## 6. About the Author / 作者について

**EN:**
I am a **retired engineer** planning this project purely as a **personal hobby (Dōraku)**. My goal is to create a **gentle, lightweight, and autonomous robot** that walks alongside people with warmth and common sense—without relying on heavy cloud servers or expensive hardware.

Building physical bipedal hardware from scratch is a bit beyond my solo hobby setup (wishing someone would give me a physical robot is half-joking, of course! 😄), so for now I am enjoying designing the software and simulation first. **I would be very happy if you find this idea interesting!**

**JA:**
私は**引退したエンジニア**で、このプロジェクトは純粋な**個人の道楽（趣味）**として取り組んでいます。巨大なサーバーや高価な機材に頼るのではなく、**「人の気持ちに寄り添う、優しくて軽量な自律ロボット」**を自分の手で作ってみたいという思いから構想を練り始めました。

1人の道楽では2足歩行のハードウェア制作までなかなか手が回らないため（※誰かロボット本体をくれないかな、というのは半分冗談です 😄）、まずはソフトウェアとシミュレーションの設計からマイペースに楽しんでいます。**このアイデアに少しでも興味を持っていただけたら嬉しいです。**

---

## ⚠️ Safety & Legal Disclaimer / 安全および法的免責事項

**EN:**
* **Experimental Concept & Software Only:** This project is provided strictly for educational, research, and simulation purposes. The author assumes **no responsibility or liability** for any physical injury, property damage, or hardware failure resulting from applying these concepts or code to physical robots. Always implement hardware-level emergency stop (kill-switch) mechanisms when testing physical actuators.
* **Public Space & Traffic Laws:** Operating autonomous robots on public streets or sidewalks is subject to local traffic laws and safety regulations. Users are solely responsible for complying with all applicable laws in their jurisdiction.
* **Privacy & Camera Usage:** When using vision/face-recognition concepts (`tapo_stereo_vision.py`), respect bystanders' privacy and portrait rights. Do not store or distribute identifiable facial data without explicit consent.

**JA:**
* **実験・シミュレーション目的の限定：** 本プロジェクトの構想およびコードは、教育・研究・シミュレーション検証を目的として公開されています。本アイデアやコードを実機のロボットハードウェアに適用したことによって生じた**いかなる怪我、器物破損、機体の故障についても、作者は一切の責任を負いません**。実機でテストする際は、必ずハードウェア側の緊急停止装置（キルスイッチ）を実装してください。
* **公道走行および法規制の遵守：** 公道や歩道などの公共空間で自律ロボットを動作させる場合、道路交通法や各自治体の条例・安全基準の適用を受けます。実験を行う際は、必ず現地の法令を遵守し、利用者の自己責任において実施してください。
* **プライバシーおよびカメラ利用の配慮：** 視覚・顔認証モジュール（`tapo_stereo_vision.py`）を使用する際は、通行人のプライバシーや肖像権に十分配慮し、本人の同意なく個人を特定できる顔データを保存・公開しないでください。

---

## License / ライセンス
MIT License
