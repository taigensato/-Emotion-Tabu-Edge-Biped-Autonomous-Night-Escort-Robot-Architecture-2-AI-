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
"""
Project Status / プロジェクトの位置づけについて
EN: This repository is a Concept Proposal and Architectural Design (Pre-Implementation / Planning Phase). While the mathematical formulation and initial draft designs for a 2D simulation have been outlined, actual implementation and testing are upcoming steps. I would be very happy if you find this idea interesting!

JA: 本リポジトリは、現在進行形の**コンセプト提案（ジャストアイデア）およびアーキテクチャ設計段階（これから実装・検証を進めるフェーズ）**です。数理モデルやPC上の2Dシミュレーションに向けた設計メモは用意していますが、実際の実装・動作確認はこれから取り組む段階です。このアイデアに少しでも興味を持っていただけたら嬉しいです。

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

