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

1. **Ultra-Lightweight Edge Execution (No PyTorch / No Unity Required)**
   * Designed to run entirely offline on an **Android Tablet (Termux + Python + NumPy)**.
   * Instead of heavy diffusion models or giant Transformers, it uses an ensemble of **Tiny MLPs (< 3KB each)** generated offline via behavior-space Tabu Search, blended smoothly by a continuous **Emotion/Temperament Vector (\(\mathbf{e}_t\))**.
   * **超軽量エッジ完結**：重い深層学習フレームワークや3Dゲームエンジンを使わず、Androidタブレット（Termux上の純粋なPython＋NumPy）のみでミリ秒単位の推論・制御を行う設計です。

2. **Dual Tabu Potential Field (Human Feedback + Fatal Fall Memory)**
   * **Soft Tabu (\(\mathcal{H}_{\text{human}}\)):** Triggered when a human frowns or says *"That's weird!"* (`それ変ダヨ`). The robot smoothly shifts its walking style (e.g., from bold strides to polite, cautious steps) via potential field repulsion.
   * **Permanent Fatal Tabu (\(\mathcal{H}_{\text{fatal}}\)):** If the robot actually falls, the temperament state that caused the fall is permanently saved (`fatal_tabu.npy`), ensuring the robot never repeats the same physical mistake on that floor surface.
   * **二重タブーポテンシャル場**：「人間からのダメ出し（一時タブー）」と「実際の転倒原因（永久タブー）」の2種類の斥力により、人間の好みに空気を合わせつつ、物理的に転倒する危険領域を永久に回避する数理モデルです。

3. **Low-Cost Night Vision Stereo & Expressive Body (`Tapo C200 × 2`)**
   * Plans to use two inexpensive consumer pan-tilt cameras (**TP-Link Tapo C200**) over local RTSP (`stream2`) for **stereo disparity 3D distance measurement**, object recognition, and **850nm IR night vision** for dark streets.
   * Plans to use the cameras' built-in pan/tilt motors (via ONVIF) for physical emotional expression (nodding happily, shaking head in confusion).
   * **市販カメラ2台による立体視・赤外線暗視・感情表現**：安価な見守りカメラ（Tapo C200）を2台並べることで、3D距離計測、深夜の赤外線暗視、さらにカメラ自体の首振り（うなずき・首振り）による感情表現を低予算で実現する計画です。

---

## 3. System Architecture / システム全体構成

```text
┌─────────────────────────────────────────────────────────────┐
│ [Layer 1: Conversational Brain] (Existing Bartender Droid)  │
│  Gemma 2B (Domain Classification) + Eliza Dict & Tabu Search│
└──────────────────────────────┬──────────────────────────────┘
                               │ Domain & Dialog Tabu (UDP 5001)
┌──────────────────────────────▼──────────────────────────────┐
│ [Layer 2: Locomotion & Emotion AI] (Termux Python + NumPy)  │
│  ・TinyMLP Gait Ensemble (Curve Walking & Stand-up)         │
│  ・Dual Tabu Potential Field (Human NG & Fatal Fall Memory) │
└───────▲──────────────────────────────────────────────┬──────┘
        │ 3D Dist / Object / Face NG (UDP 5001)        │ 7-DOF Joints (UDP 5002)
┌───────┴──────────────────────┐ ┌─────────────────────▼──────┐
│ [Vision: Dual Tapo C200]     │ │ [Layer 3: Robot OS / MCU]  │
│  ・IR Night Stereo Distance  │ │  ・1000Hz IMU Safety Filter│
│  ・Pan/Tilt Emotion Express  │ │  ・Emergency Limp & Standup│
└──────────────────────────────┘ └────────────────────────────┘
```

---

## 4. Planned Modules & Prototype Blueprints / 実装予定のモジュール構成

* `train_system.py` *(Draft/Planned)* — Offline Tabu Search generator for diverse curved-walking gaits and autonomous stand-up policies (`*.npz`). / 多様なカーブ歩行気質と自律起き上がり戦略を事前生成する学習スクリプト（設計・実装予定）。
* `return_home_brain.py` *(Draft/Planned)* — Main 50Hz Termux control loop integrating dialog signals, vision packets, and dual-tabu emotion updates. / 対話・視覚・OSフィードバックを統合するメイン歩行脳スクリプト（設計・実装予定）。
* `virtual_town_os.py` *(Draft/Planned)* — Standalone PC 2D simulator (`Tkinter`) to test dark/slippery streets, passersby giving directions, safety overrides, and fall recovery before real-world testing. / 実機テスト前にPC上で街並み・通行人・転倒復帰を検証するための模擬ロボットOSシミュレータ（設計・実装予定）。
* `tapo_stereo_vision.py` *(Draft/Planned)* — Dual Tapo C200 RTSP stereo distance estimator, face/emotion detector, and ONVIF pan/tilt gesture controller. / Tapo C200（2台）によるステレオ測距・顔認識・首振り感情表現スクリプト（設計・実装予定）。

---

## 5. Planned Workflow & Simulation Test / 今後の実装・検証ステップ

Once the initial prototype scripts are implemented, testing is planned on a PC in the following sequence:
これから実装を進めるにあたり、まずはPC上の仮想環境で以下の手順による検証を予定しています。

```bash
# 1. Install minimal dependencies / 最小依存パッケージの導入
pip install numpy opencv-python onvif-zeep

# 2. Generate diverse walking & stand-up policies via Tabu Search / タブーサーチで気質モデルを生成
python train_system.py

# 3. Launch the Virtual Town & Robot OS Simulator (Terminal 1) / 仮想街並みOSを起動
python virtual_town_os.py

# 4. Launch the Emotion-Tabu Walking Brain (Terminal 2) / 歩行脳を起動
python return_home_brain.py
```

### Planned Interactive Test Keys / シミュレータで予定している検証操作
* `[SPACE]` : Simulate a passerby making a *"That's weird!"* (NG) face (triggers Soft Tabu). / 通行人の「それ変ダヨ！」顔を模擬入力（一時タブー追加）
* `[F]` : Force a fall to test emergency limp, `fatal_tabu.npy` recording, and autonomous stand-up. / 強制転倒テスト（永久タブーの記録と自律起き上がりの検証）
* `[Left / Right Arrow]` : Simulate conversational direction guidance from the bartender droid. / 会話エンジンからの左右方向指示を模擬送信

---

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
