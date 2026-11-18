<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>人狼ゲーム in US</title>
    <style>
        body { font-family: sans-serif; background: #1e2022; color: #fff; text-align: center; padding: 20px; margin: 0; }
        .card { background: #2c3034; padding: 20px; border-radius: 12px; max-width: 540px; margin: 20px auto; box-shadow: 0 6px 12px rgba(0,0,0,0.4); text-align: left; }
        .center { text-align: center; }
        button { background: #ff4757; color: white; border: none; padding: 13px 20px; font-size: 16px; border-radius: 6px; cursor: pointer; margin-top: 12px; display: block; width: 100%; font-weight: bold; }
        button:hover { background: #ff6b81; }
        button.secondary { background: #57606f; }
        button.secondary:hover { background: #747d8c; }
        button.night-btn { background: #3742fa; }
        button.night-btn:hover { background: #5352ed; }
        button.action-target { background: #1e2022; border: 1px solid #57606f; text-align: left; padding-left: 20px; }
        button.action-target:hover { background: #ff4757; border-color: #ff4757; }
        input[type="text"] { width: 100%; padding: 10px; font-size: 16px; margin: 5px 0 15px 0; border-radius: 6px; border: none; box-sizing: border-box; }
        
        .stepper { display: inline-flex; align-items: center; gap: 6px; flex-shrink: 0; margin-left: 8px; }
        .stepper button { width: 32px; height: 32px; padding: 0; margin: 0; font-size: 18px; line-height: 1; border-radius: 6px; background: #57606f; display: inline-flex; align-items: center; justify-content: center; }
        .stepper button:hover:not(:disabled) { background: #ff4757; }
        .stepper button:disabled { opacity: 0.3; cursor: not-allowed; }
        .stepper span { min-width: 24px; text-align: center; font-weight: bold; font-size: 16px; }

        .total-banner { background: #1e2022; border: 1px solid #57606f; padding: 10px; border-radius: 8px; text-align: center; margin-bottom: 12px; font-size: 15px; font-weight: bold; }
        .total-ok { border-color: #2ed573; color: #2ed573; }
        .total-warn { border-color: #ffa502; color: #ffa502; }

        .role-section { margin-bottom: 14px; }
        .role-section h4 { margin: 5px 0 8px 0; color: #ff6b81; border-bottom: 1px solid #444; padding-bottom: 3px; }
        .role-row { display: flex; align-items: center; justify-content: space-between; margin: 10px 0; font-size: 15px; line-height: 1.4; }
        .role-left { display: flex; align-items: flex-start; gap: 8px; cursor: pointer; flex: 1; }
        .role-left input[type="checkbox"] { margin-top: 4px; width: 16px; height: 16px; flex-shrink: 0; }
        .role-desc { font-size: 12px; color: #bbb; display: block; margin-top: 2px; }
        .info-box { font-size: 15px; color: #eee; background: #1e2022; padding: 14px; border-radius: 8px; margin: 14px 0; line-height: 1.6; text-align: left; }
        .mode-box { background: #ff4757; padding: 12px; border-radius: 8px; margin-bottom: 15px; text-align: center; }
        .timer-box { background: #222; border: 2px solid #3742fa; padding: 12px; border-radius: 8px; margin-bottom: 15px; display: flex; justify-content: space-between; align-items: center; }
        .hidden { display: none !important; }
        .status-dead { text-decoration: line-through; color: #888; }
        .badge-dead { background: #555; color: #ccc; font-size: 12px; padding: 3px 8px; border-radius: 4px; }
        .badge-alive { background: #2ed573; color: #111; font-size: 12px; padding: 3px 8px; border-radius: 4px; font-weight: bold; }
        .sudden-death-banner { background: #ffa502; color: #111; font-weight: bold; padding: 8px; border-radius: 6px; margin-bottom: 12px; }
    </style>
</head>
<body>

    <!-- 1. 初期設定画面 -->
    <div id="setup-screen" class="card">
        <h2 class="center">🐺 人狼ゲーム in US 🇺🇸</h2>
        
        <label>参加者の名前（カンマ区切り）:</label>
        <input type="text" id="names" value="A君, B君, C君, D君, E君" oninput="onNamesChange()">

        <div class="mode-box">
            <label style="cursor: pointer; font-weight: bold; font-size: 16px; display: inline-flex; align-items: center; gap: 8px;">
                <input type="checkbox" id="mode-random" onchange="toggleMode()" style="width: 18px; height: 18px;">
                🎲 チェックした役職の中からランダムに選ぶ
            </label>
        </div>

        <div id="total-counter-banner" class="total-banner"></div>

        <div class="role-section">
            <h4>🐺 人狼陣営</h4>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-wolf" checked> <span><b>人狼 🐺</b><span class="role-desc">毎晩1人を襲撃。人間と同数以下で勝利。</span></span></label>
                <div class="stepper" data-key="wolf">
                    <button type="button" class="btn-minus" onclick="changeCount('wolf', -1)">－</button>
                    <span id="count-wolf">1</span>
                    <button type="button" class="btn-plus" onclick="changeCount('wolf', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-madman" checked> <span><b>狂人 🤪</b><span class="role-desc">人間カウントだが人狼の味方。</span></span></label>
                <div class="stepper" data-key="madman">
                    <button type="button" class="btn-minus" onclick="changeCount('madman', -1)">－</button>
                    <span id="count-madman">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('madman', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-cultist"> <span><b>狂信者 🧠</b><span class="role-desc">最初から誰が人狼か知っている。</span></span></label>
                <div class="stepper" data-key="cultist">
                    <button type="button" class="btn-minus" onclick="changeCount('cultist', -1)">－</button>
                    <span id="count-cultist">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('cultist', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-blackcat"> <span><b>黒猫 🐈‍⬛</b><span class="role-desc">処刑されると生存者から1人道連れ。</span></span></label>
                <div class="stepper" data-key="blackcat">
                    <button type="button" class="btn-minus" onclick="changeCount('blackcat', -1)">－</button>
                    <span id="count-blackcat">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('blackcat', 1)">＋</button>
                </div>
            </div>
        </div>

        <div class="role-section">
            <h4>🔮 人間陣営</h4>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-villager" checked> <span><b>村人 👤</b><span class="role-desc">特殊能力なし。推理で人狼を探す。</span></span></label>
                <div class="stepper" data-key="villager">
                    <button type="button" class="btn-minus" onclick="changeCount('villager', -1)">－</button>
                    <span id="count-villager">3</span>
                    <button type="button" class="btn-plus" onclick="changeCount('villager', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-seer" checked> <span><b>占い師 🔮</b><span class="role-desc">毎晩1人を占い人狼か判定する。</span></span></label>
                <div class="stepper" data-key="seer">
                    <button type="button" class="btn-minus" onclick="changeCount('seer', -1)">－</button>
                    <span id="count-seer">1</span>
                    <button type="button" class="btn-plus" onclick="changeCount('seer', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-medium" checked> <span><b>霊能者 👻</b><span class="role-desc">処刑された人が人狼か判定する。</span></span></label>
                <div class="stepper" data-key="medium">
                    <button type="button" class="btn-minus" onclick="changeCount('medium', -1)">－</button>
                    <span id="count-medium">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('medium', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-hunter" checked> <span><b>騎士 🛡️</b><span class="role-desc">毎晩1人を守り襲撃を防ぐ。</span></span></label>
                <div class="stepper" data-key="hunter">
                    <button type="button" class="btn-minus" onclick="changeCount('hunter', -1)">－</button>
                    <span id="count-hunter">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('hunter', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-trapper"> <span><b>罠師 🪤</b><span class="role-desc">1度だけ自宅に罠を設置。次の夜に訪れた人を返り討ちにする（人狼複数ならランダム1人死亡）。</span></span></label>
                <div class="stepper" data-key="trapper">
                    <button type="button" class="btn-minus" onclick="changeCount('trapper', -1)">－</button>
                    <span id="count-trapper">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('trapper', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-drunk"> <span><b>酔っぱらい 🍺</b><span class="role-desc">1日目は他人の役職と思い込んで行動。2日目の夜に村人だと気づく。</span></span></label>
                <div class="stepper" data-key="drunk">
                    <button type="button" class="btn-minus" onclick="changeCount('drunk', -1)">－</button>
                    <span id="count-drunk">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('drunk', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-baker"> <span><b>パン屋さん 🥖</b><span class="role-desc">生存中、毎朝美味しいパンが焼ける。</span></span></label>
                <div class="stepper" data-key="baker">
                    <button type="button" class="btn-minus" onclick="changeCount('baker', -1)">－</button>
                    <span id="count-baker">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('baker', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-couple"> <span><b>カップル 💑</b><span class="role-desc">（※2人セット）結ばれた2人。</span></span></label>
                <div class="stepper" data-key="couple">
                    <button type="button" class="btn-minus" onclick="changeCount('couple', -2)">－</button>
                    <span id="count-couple">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('couple', 2)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-nekomata"> <span><b>猫又 🐾</b><span class="role-desc">襲撃者や処刑者を道連れにする。</span></span></label>
                <div class="stepper" data-key="nekomata">
                    <button type="button" class="btn-minus" onclick="changeCount('nekomata', -1)">－</button>
                    <span id="count-nekomata">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('nekomata', 1)">＋</button>
                </div>
            </div>
        </div>

        <div class="role-section">
            <h4>🦊 第3陣営・特殊陣営</h4>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-fox"> <span><b>狐 🦊</b><span class="role-desc">襲撃死なし・占い死あり・生存勝利。</span></span></label>
                <div class="stepper" data-key="fox">
                    <button type="button" class="btn-minus" onclick="changeCount('fox', -1)">－</button>
                    <span id="count-fox">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('fox', 1)">＋</button>
                </div>
            </div>
            <div class="role-row">
                <label class="role-left"><input type="checkbox" id="role-teruteru"> <span><b>てるてる ☀️</b><span class="role-desc">判定は人間側。昼の会議の投票で処刑されると単独勝利！</span></span></label>
                <div class="stepper" data-key="teruteru">
                    <button type="button" class="btn-minus" onclick="changeCount('teruteru', -1)">－</button>
                    <span id="count-teruteru">0</span>
                    <button type="button" class="btn-plus" onclick="changeCount('teruteru', 1)">＋</button>
                </div>
            </div>
            <div class="role-row" style="background: rgba(255, 71, 87, 0.12); padding: 8px; border-radius: 6px; border: 1px dashed #ff6b81;">
                <label style="display: flex; align-items: flex-start; gap: 8px; cursor: pointer; flex: 1;">
                    <input type="checkbox" id="add-lovers" style="margin-top: 4px; width: 18px; height: 18px; flex-shrink: 0;">
                    <span><b>恋人 💘（追加役職・人数カウント外）</b><span class="role-desc">参加者からランダム2人に追加付与。お互いの名前と役職を知る。異なる陣営同士なら第3陣営として2人での勝利を目指す！</span></span>
                </label>
            </div>
        </div>

        <button onclick="goToRuleSettings()">次へ（特殊ルール設定）</button>
    </div>

    <!-- 1.5 特殊ルール設定画面 -->
    <div id="rule-screen" class="card center hidden">
        <h2>⚙️ 特殊ルール設定</h2>
        <p style="color: #aaa; font-size: 14px;">このゲームで採用するルールを選んでください。</p>
        
        <div class="role-section" style="margin-top: 20px;">
            <label style="display: flex; align-items: center; justify-content: space-between; margin: 12px 0; font-size: 16px; cursor: pointer;">
                <span>🔮 占い師へのお告げ（初日に人間側の安全な人が1人分かる）</span>
                <input type="checkbox" id="rule-oracle" style="width: 20px; height: 20px;">
            </label>
            <label style="display: flex; align-items: center; justify-content: space-between; margin: 12px 0; font-size: 16px; cursor: pointer;">
                <span>⏩ 初日の昼投票スキップ（少人数向け。初日の議論後に投票せずすぐ夜へ）</span>
                <input type="checkbox" id="rule-skip-first-vote" style="width: 20px; height: 20px;">
            </label>
            <label style="display: flex; align-items: center; justify-content: space-between; margin: 12px 0; font-size: 16px; cursor: pointer;">
                <span>🛡️ 騎士の自分ガード（自分自身を護衛できる）</span>
                <input type="checkbox" id="rule-self-guard" style="width: 20px; height: 20px;">
            </label>
            <label style="display: flex; align-items: center; justify-content: space-between; margin: 12px 0; font-size: 16px; cursor: pointer;">
                <span>🛡️ 騎士の連続ガード（同じ人を2晩連続で守れる）</span>
                <input type="checkbox" id="rule-continuous-guard" style="width: 20px; height: 20px;">
            </label>
            <label style="display: flex; align-items: center; justify-content: space-between; margin: 12px 0; font-size: 16px; cursor: pointer;">
                <span>💔 カップル・恋人の後追い自殺（片方が死ぬともう1人も殉死）</span>
                <input type="checkbox" id="rule-lover-suicide" style="width: 20px; height: 20px;">
            </label>
            <label style="display: flex; align-items: center; justify-content: space-between; margin: 12px 0; font-size: 16px; cursor: pointer;">
                <span>🌀 混乱モード（人狼以外の誰か1人だけ役職が不明。2日目の夜に判明）</span>
                <input type="checkbox" id="rule-confusion" style="width: 20px; height: 20px;">
            </label>
        </div>

        <button onclick="startGameSetup()">役職を決定する</button>
        <button class="secondary" onclick="backToSetup()">戻る</button>
    </div>

    <!-- 2. 役職確認画面 -->
    <div id="check-screen" class="card center hidden">
        <h2 id="current-player-title">役職確認</h2>
        <p id="instruction">他の人に見えないようにスマホを渡してください</p>
        <button id="show-btn" onclick="toggleRole()">自分の役職を見る</button>
        <div id="role-display" class="hidden" style="margin: 20px 0;">
            <div id="role-name-text" style="font-size: 24px; color: #ff4757; font-weight: bold;"></div>
            <div id="role-sub-text" style="font-size: 18px; color: #ff6b81; font-weight: bold; margin-top: 6px;"></div>
            <div id="role-ally-text" style="font-size: 15px; color: #ffa502; font-weight: bold; margin-top: 8px; line-height: 1.5;"></div>
            <div id="role-desc-text" class="info-box"></div>
        </div>
        <button id="next-btn" class="hidden" onclick="nextPlayer()">確認完了（次の人へ）</button>
    </div>

    <!-- 3. 昼の議論メイン画面 -->
    <div id="game-screen" class="card center hidden">
        <h2 id="day-title">1日目・昼の議論</h2>
        
        <div class="timer-box">
            <div>
                <div style="font-size: 12px; color: #aaa;">残り議論時間</div>
                <div id="timer-display" style="font-size: 24px; font-weight: bold; color: #2ed573;">03:00</div>
            </div>
            <div style="display: flex; align-items: center; gap: 8px;">
                <button type="button" onclick="adjustTimer(-60)" style="width: 36px; height: 36px; margin: 0; padding: 0;">－</button>
                <button type="button" onclick="adjustTimer(60)" style="width: 36px; height: 36px; margin: 0; padding: 0;">＋</button>
            </div>
        </div>

        <div id="morning-news" class="info-box hidden"></div>
        
        <div id="fixed-composition-box" class="info-box hidden" style="border: 1px solid #ff4757;">
            <div style="font-weight: bold; color: #ff6b81; margin-bottom: 6px;">📜 本日の村の役職構成（内訳）</div>
            <div id="fixed-composition-list"></div>
        </div>

        <div id="vote-action-container">
            <p>議論が終わったら投票に進んでください。</p>
            <button onclick="startVoting(null)">🗳️ 投票を始める（1人ずつ）</button>
        </div>

        <hr style="border-color: #444; margin: 20px 0;">
        <h3>現在の生存状況</h3>
        <ul id="player-list" style="list-style: none; padding: 0; text-align: left;"></ul>

        <button id="toggle-gm-btn" class="secondary" onclick="toggleGmRoles()">👁️ GM用：全役職を表示/隠す</button>
        <button class="secondary" onclick="confirmReset()">🔄 最初からやり直す</button>
    </div>

    <!-- 4. 秘密投票画面 -->
    <div id="vote-screen" class="card center hidden">
        <div id="sudden-death-label" class="sudden-death-banner hidden">⚡ 決選投票（サドンデス）⚡</div>
        <h2 id="voter-title">投票タイム</h2>
        <p id="vote-instruction">他の人に見えないようにスマホを渡してください</p>
        
        <button id="open-vote-btn" onclick="openVoteOptions()">投票画面を開く</button>
        
        <div id="vote-options-area" class="hidden">
            <p style="color: #ff6b81; font-weight: bold;">処刑したい人を1人選んでタップしてください：</p>
            <div id="vote-buttons-list"></div>
        </div>
    </div>

    <!-- 5. 投票結果発表画面 -->
    <div id="vote-result-screen" class="card center hidden">
        <h2>📊 開票結果</h2>
        <div id="vote-summary" class="info-box"></div>
        <div id="execution-announcement" style="font-size: 18px; font-weight: bold; margin: 15px 0; color: #ff4757;"></div>
        <div id="execution-extra-news" style="font-size: 15px; color: #ffa502; margin-bottom: 12px;"></div>
        
        <button id="to-night-btn" class="night-btn" onclick="proceedToNight()">🌙 夜の行動へ進む</button>
        <button id="revote-btn" onclick="startSuddenDeath()">⚡ サドンデス投票へ進む</button>
    </div>

    <!-- 6. 夜の行動画面 -->
    <div id="night-screen" class="card center hidden">
        <h2 id="night-player-title">🌙 夜の行動</h2>
        <p id="night-instruction">他の人に見えないようにスマホを渡してください</p>
        
        <button id="open-night-btn" class="night-btn" onclick="openNightAction()">夜の画面を開く</button>
        
        <div id="night-action-area" class="hidden">
            <div id="night-role-info" class="info-box"></div>
            <div id="night-targets-list"></div>
            <button id="night-done-btn" class="hidden" onclick="nextNightPlayer()">行動完了（次の人へ）</button>
        </div>
    </div>

    <!-- 7. ゲーム終了画面 -->
    <div id="gameover-screen" class="card center hidden">
        <h1 id="winner-title" style="color: #ffa502;">🏆 ゲーム終了！</h1>
        <p id="winner-reason" style="font-size: 16px; line-height: 1.6;"></p>
        <hr style="border-color: #444; margin: 20px 0;">
        <h3>最終結果・役職一覧</h3>
        <ul id="final-player-list" style="list-style: none; padding: 0; text-align: left;"></ul>
        <button onclick="resetToSetupScreen(true)">🔄 もう一度遊ぶ（設定を維持）</button>
        <button class="secondary" onclick="resetToSetupScreen(false)">⚙️ 初期設定に戻して遊ぶ（デフォルト）</button>
    </div>

    <script>
        const ROLE_DESCRIPTIONS = {
            '人狼 🐺': '【人狼陣営】毎晩1人を襲撃できます。人間陣営を人狼と同数以下まで減らせば勝利です。',
            '狂人 🤪': '【人狼陣営】特殊能力はありませんが、人狼が勝つと勝利です。占い結果は「人間」と出ます。',
            '狂信者 🧠': '【人狼陣営】最初から誰が人狼かを知っている狂人です。人狼をサポートしましょう。',
            '黒猫 🐈‍⬛': '【人狼陣営】人狼の味方です。昼に処刑されると、生存者から1人道連れにします。',
            '村人 👤': '【人間陣営】特殊能力はありません。推理と議論で人狼を見つけ出しましょう。',
            '占い師 🔮': '【人間陣営】毎晩1人を占い、「人狼」か「人間」かを知ることができます。（狐を占うと呪殺）',
            '霊能者 👻': '【人間陣営】前の日の昼に処刑された人が「人狼」だったか「人間」だったかを知ることができます。',
            '騎士 🛡️': '【人間陣営】毎晩1人を守り、人狼の襲撃から救うことができます。',
            '罠師 🪤': '【人間陣営】1ゲームに1回、夜に自分の家へ罠を設置できます。設置した次の夜に訪れた人を返り討ちにします（複数人狼ならランダム1人死亡）。',
            '酔っぱらい 🍺': '【人間陣営】1日目は酔っているため別の役職だと思い込んで行動します。2日目の夜に酔いが覚めて自分が村人だと気づきます。',
            'パン屋さん 🥖': '【人間陣営】生存している限り、毎朝美味しいパンが焼けたことが村全体に知らされます。',
            'カップル 💑': '【人間陣営】もう1人のカップルとお互いに結ばれています。',
            '猫又 🐾': '【人間陣営】夜に襲撃されると人狼を1匹道連れにします。（昼に処刑されるとランダム道連れ）',
            '狐 🦊': '【第3陣営】襲撃死しませんが、占い師に占われると死亡します。決着時に生き残っていれば単独勝利！',
            'てるてる ☀️': '【第3陣営】判定は人間側ですが、昼の会議の投票で処刑された場合のみ単独勝利となります！生存していても村側が勝っても負けになります。'
        };

        const roleKeyMap = {
            '人狼 🐺': 'wolf',
            '狂人 🤪': 'madman',
            '狂信者 🧠': 'cultist',
            '黒猫 🐈‍⬛': 'blackcat',
            '村人 👤': 'villager',
            '占い師 🔮': 'seer',
            '霊能者 👻': 'medium',
            '騎士 🛡️': 'hunter',
            '罠師 🪤': 'trapper',
            '酔っぱらい 🍺': 'drunk',
            'パン屋さん 🥖': 'baker',
            'カップル 💑': 'couple',
            '猫又 🐾': 'nekomata',
            '狐 🦊': 'fox',
            'てるてる ☀️': 'teruteru'
        };

        let roleCounts = {
            wolf: 1, madman: 0, cultist: 0, blackcat: 0,
            villager: 3, seer: 1, medium: 0, hunter: 0,
            trapper: 0, drunk: 0,
            baker: 0, couple: 0, nekomata: 0, fox: 0, teruteru: 0
        };

        let gameRules = {
            oracle: false,
            skipFirstVote: false,
            selfGuard: false,
            continuousGuard: false,
            loverSuicide: false,
            confusion: false
        };

        let playerData = [];
        let currentIndex = 0;
        let showGmRoles = false;
        let dayCount = 1;
        let isFixedMode = true;

        let timerSeconds = 180;
        let timerInterval = null;

        let alivePlayers = [];
        let voterIndex = 0;
        let voteCounts = {};
        let suddenDeathCandidateIndices = null;
        let lastExecutedPlayer = null;
        let lastGuardedIndex = null;

        let nightActorIndex = 0;
        let wolfVotes = {};
        let nightVisits = [];
        let foxKilledIndex = null;
        let confusedPlayerIndex = null;
        let trapKillTriggeredByWolves = false; // 人狼が罠で全滅したかどうか

        window.onload = () => {
            toggleMode();
        };

        function getPlayerNames() {
            const input = document.getElementById('names').value;
            return input.split(',').map(s => s.trim()).filter(s => s !== '');
        }

        function getTotalRoleCount() {
            let sum = 0;
            for (let k in roleCounts) sum += roleCounts[k];
            return sum;
        }

        function toggleMode() {
            const isRandom = document.getElementById('mode-random').checked;
            isFixedMode = !isRandom;
            const steppers = document.querySelectorAll('.stepper');
            const checkboxes = document.querySelectorAll('.role-left input[type="checkbox"]');
            const banner = document.getElementById('total-counter-banner');

            if (isRandom) {
                steppers.forEach(s => s.classList.add('hidden'));
                checkboxes.forEach(chk => {
                    if (chk.id !== 'add-lovers') chk.classList.remove('hidden');
                });
                banner.classList.add('hidden');
            } else {
                steppers.forEach(s => s.classList.remove('hidden'));
                checkboxes.forEach(chk => {
                    if (chk.id !== 'add-lovers') chk.classList.add('hidden');
                });
                banner.classList.remove('hidden');
                syncCountsWithPlayerCount();
            }
        }

        function onNamesChange() {
            if (!document.getElementById('mode-random').checked) {
                syncCountsWithPlayerCount();
            }
        }

        function syncCountsWithPlayerCount() {
            const targetTotal = Math.max(getPlayerNames().length, 1);
            let currentTotal = getTotalRoleCount();

            while (currentTotal > targetTotal) {
                if (roleCounts.villager > 0) {
                    roleCounts.villager--;
                } else {
                    const keys = Object.keys(roleCounts).reverse();
                    for (let k of keys) {
                        const minVal = 0;
                        const step = (k === 'couple') ? 2 : 1;
                        if (roleCounts[k] - step >= minVal) {
                            roleCounts[k] -= step;
                            break;
                        }
                    }
                }
                currentTotal = getTotalRoleCount();
                if (currentTotal <= 0) break;
            }
            updateStepperUI();
        }

        function changeCount(key, delta) {
            const maxPlayers = getPlayerNames().length;
            const currentTotal = getTotalRoleCount();
            const minVal = 0;

            const nextVal = roleCounts[key] + delta;
            const nextTotal = currentTotal + delta;

            if (nextVal < minVal) return;
            if (nextTotal > maxPlayers) return;

            roleCounts[key] = nextVal;
            updateStepperUI();
        }

        function updateStepperUI() {
            const maxPlayers = getPlayerNames().length;
            const currentTotal = getTotalRoleCount();
            const remaining = maxPlayers - currentTotal;

            const banner = document.getElementById('total-counter-banner');
            if (remaining === 0) {
                banner.className = 'total-banner total-ok';
                banner.innerText = `✅ 役職合計： ${currentTotal} / ${maxPlayers} 人（ピッタリ！）`;
            } else if (remaining > 0) {
                banner.className = 'total-banner total-warn';
                banner.innerText = `⚠️ 役職合計： ${currentTotal} / ${maxPlayers} 人（あと ${remaining} 人分必要）`;
            } else {
                banner.className = 'total-banner total-warn';
                banner.innerText = `⚠️ 役職合計： ${currentTotal} / ${maxPlayers} 人（${Math.abs(remaining)} 人オーバー）`;
            }

            document.querySelectorAll('.stepper').forEach(stepper => {
                const key = stepper.getAttribute('data-key');
                const val = roleCounts[key];
                const minVal = 0;
                const step = (key === 'couple') ? 2 : 1;

                stepper.querySelector('span').innerText = val;
                stepper.querySelector('.btn-minus').disabled = (val - step < minVal);
                stepper.querySelector('.btn-plus').disabled = (currentTotal + step > maxPlayers);
            });
        }

        function goToRuleSettings() {
            const names = getPlayerNames();
            if (names.length < 3) {
                alert('3人以上でプレイしてください！');
                return;
            }
            const isRandomMode = document.getElementById('mode-random').checked;
            if (!isRandomMode) {
                const totalSelected = getTotalRoleCount();
                if (totalSelected !== names.length) {
                    alert(`参加人数は ${names.length}人 ですが、役職の合計が ${totalSelected}人 になっています！`);
                    return;
                }
            }

            document.getElementById('setup-screen').classList.add('hidden');
            document.getElementById('rule-screen').classList.remove('hidden');
        }

        function backToSetup() {
            document.getElementById('rule-screen').classList.add('hidden');
            document.getElementById('setup-screen').classList.remove('hidden');
        }

        function startGameSetup() {
            gameRules.oracle = document.getElementById('rule-oracle').checked;
            gameRules.skipFirstVote = document.getElementById('rule-skip-first-vote').checked;
            gameRules.selfGuard = document.getElementById('rule-self-guard').checked;
            gameRules.continuousGuard = document.getElementById('rule-continuous-guard').checked;
            gameRules.loverSuicide = document.getElementById('rule-lover-suicide').checked;
            gameRules.confusion = document.getElementById('rule-confusion').checked;

            const names = getPlayerNames();
            let roles = [];
            const isRandomMode = document.getElementById('mode-random').checked;
            isFixedMode = !isRandomMode;

            if (isRandomMode) {
                let candidateRoles = [];
                if (document.getElementById('role-wolf').checked) candidateRoles.push('人狼 🐺');
                if (document.getElementById('role-madman').checked) candidateRoles.push('狂人 🤪');
                if (document.getElementById('role-cultist').checked) candidateRoles.push('狂信者 🧠');
                if (document.getElementById('role-blackcat').checked) candidateRoles.push('黒猫 🐈‍⬛');
                if (document.getElementById('role-villager').checked) candidateRoles.push('村人 👤');
                if (document.getElementById('role-seer').checked) candidateRoles.push('占い師 🔮');
                if (document.getElementById('role-medium').checked) candidateRoles.push('霊能者 👻');
                if (document.getElementById('role-hunter').checked) candidateRoles.push('騎士 🛡️');
                if (document.getElementById('role-trapper').checked) candidateRoles.push('罠師 🪤');
                if (document.getElementById('role-drunk').checked) candidateRoles.push('酔っぱらい 🍺');
                if (document.getElementById('role-baker').checked) candidateRoles.push('パン屋さん 🥖');
                if (document.getElementById('role-nekomata').checked) candidateRoles.push('猫又 🐾');
                if (document.getElementById('role-fox').checked) candidateRoles.push('狐 🦊');
                if (document.getElementById('role-teruteru').checked) candidateRoles.push('てるてる ☀️');

                let hasCouple = document.getElementById('role-couple').checked;
                roles = [];
                if (hasCouple) roles.push('カップル 💑', 'カップル 💑');

                let remainingSlots = names.length - roles.length;
                candidateRoles.sort(() => Math.random() - 0.5);
                roles = roles.concat(candidateRoles.slice(0, remainingSlots));

                while (roles.length < names.length) {
                    roles.push('村人 👤');
                }
            } else {
                for (let rName in roleKeyMap) {
                    const key = roleKeyMap[rName];
                    const count = roleCounts[key];
                    for (let i = 0; i < count; i++) roles.push(rName);
                }
            }

            roles.sort(() => Math.random() - 0.5);

            playerData = names.map((name, idx) => ({
                name: name,
                role: roles[idx],
                isLover: false,
                alive: true,
                confusionRevealed: false,
                oracleMsg: '',
                trapState: 'unused',
                fakeRole: null,
                drunkSobered: false
            }));

            playerData.forEach((p, idx) => {
                if (p.role === '酔っぱらい 🍺') {
                    const otherRoles = playerData
                        .filter((_, otherIdx) => otherIdx !== idx && playerData[otherIdx].role !== '酔っぱらい 🍺')
                        .map(other => other.role);
                    if (otherRoles.length > 0) {
                        p.fakeRole = otherRoles[Math.floor(Math.random() * otherRoles.length)];
                    } else {
                        p.fakeRole = '村人 👤';
                    }
                }
            });

            const addLovers = document.getElementById('add-lovers').checked;
            if (addLovers && playerData.length >= 2) {
                let indices = playerData.map((_, i) => i);
                indices.sort(() => Math.random() - 0.5);
                playerData[indices[0]].isLover = true;
                playerData[indices[1]].isLover = true;
            }

            confusedPlayerIndex = null;
            if (gameRules.confusion) {
                const nonWolfIndices = playerData
                    .map((p, i) => ({ role: p.role, index: i }))
                    .filter(p => p.role !== '人狼 🐺')
                    .map(p => p.index);

                if (nonWolfIndices.length > 0) {
                    confusedPlayerIndex = nonWolfIndices[Math.floor(Math.random() * nonWolfIndices.length)];
                }
            }

            if (gameRules.oracle) {
                playerData.forEach((p, idx) => {
                    const effectiveRole = (p.role === '酔っぱらい 🍺' && !p.drunkSobered) ? p.fakeRole : p.role;
                    if (effectiveRole === '占い師 🔮') {
                        const otherPlayers = playerData.filter((_, otherIdx) => otherIdx !== idx);
                        if (otherPlayers.length > 0) {
                            const target = otherPlayers[Math.floor(Math.random() * otherPlayers.length)];
                            const isWolf = (p.role === '酔っぱらい 🍺') ? (Math.random() < 0.5) : (target.role === '人狼 🐺');
                            const sideText = isWolf ? '【人狼側】' : '【人間側】';
                            p.oracleMsg = `🔮 【お告げ】神託により、${target.name} さんは ${sideText} であることが分かりました！`;
                        }
                    }
                });
            }

            currentIndex = 0;
            dayCount = 1;
            lastExecutedPlayer = null;
            lastGuardedIndex = null;
            trapKillTriggeredByWolves = false;

            document.getElementById('rule-screen').classList.add('hidden');
            document.getElementById('check-screen').classList.remove('hidden');
            updateCheckScreen();
        }

        function isHumanSide(role) {
            return (
                role === '村人 👤' ||
                role === '占い師 🔮' ||
                role === '霊能者 👻' ||
                role === '騎士 🛡️' ||
                role === '罠師 🪤' ||
                role === '酔っぱらい 🍺' ||
                role === 'パン屋さん 🥖' ||
                role === 'カップル 💑' ||
                role === '猫又 🐾' ||
                role === 'てるてる ☀️'
            );
        }

        function updateCheckScreen() {
            const p = playerData[currentIndex];
            document.getElementById('current-player-title').innerText = `${p.name} さんの番です`;
            
            let isConfusedNow = (gameRules.confusion && currentIndex === confusedPlayerIndex && !p.confusionRevealed && dayCount === 1);
            let shownRole = (p.role === '酔っぱらい 🍺' && !p.drunkSobered) ? p.fakeRole : p.role;

            const displayRoleName = isConfusedNow ? '？？？（混乱中）' : shownRole;
            document.getElementById('role-name-text').innerText = `あなたの役職： ${displayRoleName}`;
            
            const subTextEl = document.getElementById('role-sub-text');
            if (p.isLover && !isConfusedNow) {
                subTextEl.innerText = '💘 追加役職：恋人';
                subTextEl.classList.remove('hidden');
            } else {
                subTextEl.innerText = '';
                subTextEl.classList.add('hidden');
            }

            let desc = isConfusedNow ? '混乱モード有効のため、あなたの役職名が隠されています。（2日目の夜に判明します）' : (ROLE_DESCRIPTIONS[shownRole] || '');
            
            let allyLines = [];
            if (!isConfusedNow) {
                if (shownRole === '人狼 🐺' || shownRole === '狂信者 🧠') {
                    const wolves = playerData.filter((other, idx) => idx !== currentIndex && other.role === '人狼 🐺').map(o => o.name);
                    if (wolves.length > 0) allyLines.push(`🤝 仲間の人狼： ${wolves.join('、')}`);
                } else if (shownRole === 'カップル 💑') {
                    const couples = playerData.filter((other, idx) => idx !== currentIndex && other.role === 'カップル 💑').map(o => o.name);
                    if (couples.length > 0) allyLines.push(`💑 あなたのカップル相手： ${couples.join('、')}`);
                } else if (shownRole === '占い師 🔮' && p.oracleMsg) {
                    allyLines.push(p.oracleMsg);
                }

                if (p.isLover) {
                    const partner = playerData.find((other, idx) => idx !== currentIndex && other.isLover);
                    if (partner) {
                        allyLines.push(`💘 あなたの恋人： ${partner.name} さん（役職：${partner.role}）`);
                        const bothHuman = isHumanSide(p.role) && isHumanSide(partner.role);
                        const bothWolf = (p.role === '人狼 🐺' && partner.role === '人狼 🐺');
                        if (bothHuman) {
                            desc += '\n\n【恋人ルール】2人とも人間陣営のため、村人陣営が勝利すれば恋人2人も勝利となります！';
                        } else if (bothWolf) {
                            desc += '\n\n【恋人ルール】2人とも人狼のため、人狼陣営として勝利を目指します！';
                        } else {
                            desc += '\n\n【恋人ルール】異なる陣営同士の恋人のため「第3陣営（恋人陣営）」となります！決着時に2人とも生き残っていれば単独勝利です！';
                        }
                    }
                }
            }

            document.getElementById('role-ally-text').innerHTML = allyLines.join('<br>');
            document.getElementById('role-desc-text').innerText = desc;
            
            document.getElementById('role-display').classList.add('hidden');
            document.getElementById('show-btn').classList.remove('hidden');
            document.getElementById('next-btn').classList.add('hidden');
        }

        function toggleRole() {
            document.getElementById('role-display').classList.remove('hidden');
            document.getElementById('show-btn').classList.add('hidden');
            document.getElementById('next-btn').classList.remove('hidden');
        }

        function nextPlayer() {
            currentIndex++;
            if (currentIndex < playerData.length) {
                updateCheckScreen();
            } else {
                document.getElementById('check-screen').classList.add('hidden');
                enterDayPhase(['ゲームが始まりました！最初の議論を行ってください。']);
            }
        }

        function startTimer() {
            if (timerInterval) clearInterval(timerInterval);
            timerSeconds = 180;
            updateTimerDisplay();
            timerInterval = setInterval(() => {
                timerSeconds--;
                updateTimerDisplay();
                if (timerSeconds <= 0) {
                    clearInterval(timerInterval);
                    alert('⏰ 議論時間が終了しました！投票に進んでください。');
                }
            }, 1000);
        }

        function adjustTimer(sec) {
            timerSeconds += sec;
            if (timerSeconds < 0) timerSeconds = 0;
            updateTimerDisplay();
        }

        function updateTimerDisplay() {
            const m = String(Math.floor(timerSeconds / 60)).padStart(2, '0');
            const s = String(timerSeconds % 60).padStart(2, '0');
            document.getElementById('timer-display').innerText = `${m}:${s}`;
        }

        function enterDayPhase(newsArray) {
            if (checkWinCondition()) return;

            document.getElementById('night-screen').classList.add('hidden');
            document.getElementById('vote-result-screen').classList.add('hidden');
            document.getElementById('game-screen').classList.remove('hidden');

            document.getElementById('day-title').innerText = `${dayCount}日目・昼の議論`;
            startTimer();

            const compBox = document.getElementById('fixed-composition-box');
            if (isFixedMode) {
                let counts = {};
                playerData.forEach(p => counts[p.role] = (counts[p.role] || 0) + 1);
                let compHtml = '';
                for (let r in counts) compHtml += `・${r}： <b>${counts[r]}人</b><br>`;
                if (playerData.some(p => p.isLover)) {
                    compHtml += `・💘 恋人（追加役職）： <b>2人</b><br>`;
                }
                document.getElementById('fixed-composition-list').innerHTML = compHtml;
                compBox.classList.remove('hidden');
            } else {
                compBox.classList.add('hidden');
            }

            if (dayCount >= 2) {
                const bakerAlive = playerData.some(p => p.alive && p.role === 'パン屋さん 🥖');
                if (bakerAlive) newsArray.push('🥖 今日もパン屋さんが美味しいパンが焼き上げました！');
            }

            const newsBox = document.getElementById('morning-news');
            if (newsArray && newsArray.length > 0) {
                newsBox.innerHTML = newsArray.map(n => `<div>${n}</div>`).join('');
                newsBox.classList.remove('hidden');
            } else {
                newsBox.classList.add('hidden');
            }

            const voteActionContainer = document.getElementById('vote-action-container');
            if (dayCount === 1 && gameRules.skipFirstVote) {
                voteActionContainer.innerHTML = `
                    <p style="color: #ffa502; font-weight: bold;">⏩ 初日投票スキップモードが有効です</p>
                    <button class="night-btn" onclick="proceedToNight()">🌙 夜の行動へ直接進む</button>
                `;
            } else {
                voteActionContainer.innerHTML = `
                    <p>議論が終わったら投票に進んでください。</p>
                    <button onclick="startVoting(null)">🗳️ 投票を始める（1人ずつ）</button>
                `;
            }

            renderPlayerList();
        }

        function renderPlayerList() {
            const list = document.getElementById('player-list');
            list.innerHTML = '';
            playerData.forEach(p => {
                const statusBadge = p.alive ? `<span class="badge-alive">生存</span>` : `<span class="badge-dead">死亡</span>`;
                const nameClass = p.alive ? '' : 'status-dead';
                const loverTag = (showGmRoles && p.isLover) ? ` <span style="color:#ffa502;">[💘恋人]</span>` : '';
                let roleExtra = '';
                if (showGmRoles && p.role === '酔っぱらい 🍺') {
                    roleExtra = ` [思い込み:${p.fakeRole}]`;
                }
                const roleText = (showGmRoles || p.confusionRevealed) ? ` <span style="color:#ff6b81;">（${p.role}${roleExtra}）</span>${loverTag}` : '';

                list.innerHTML += `
                    <li style="padding: 10px; border-bottom: 1px solid #444; display: flex; justify-content: space-between; align-items: center;">
                        <span class="${nameClass}">${p.name}${roleText}</span>
                        ${statusBadge}
                    </li>`;
            });
        }

        function toggleGmRoles() {
            showGmRoles = !showGmRoles;
            renderPlayerList();
        }

        function confirmReset() {
            if (confirm('ゲームを最初からやり直しますか？')) {
                location.reload();
            }
        }

        function resetToSetupScreen(keepSettings) {
            if (timerInterval) clearInterval(timerInterval);
            document.getElementById('gameover-screen').classList.add('hidden');

            if (keepSettings) {
                document.getElementById('setup-screen').classList.remove('hidden');
            } else {
                location.reload();
            }
        }

        function startVoting(candidatesFilterIndices) {
            if (timerInterval) clearInterval(timerInterval);
            suddenDeathCandidateIndices = candidatesFilterIndices;

            alivePlayers = playerData.map((p, idx) => ({ ...p, originalIndex: idx })).filter(p => p.alive);
            voterIndex = 0;
            voteCounts = {};

            const targetIndices = suddenDeathCandidateIndices ? suddenDeathCandidateIndices : alivePlayers.map(p => p.originalIndex);
            targetIndices.forEach(idx => voteCounts[idx] = 0);

            const sdBanner = document.getElementById('sudden-death-label');
            if (suddenDeathCandidateIndices) sdBanner.classList.remove('hidden');
            else sdBanner.classList.add('hidden');

            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('vote-result-screen').classList.add('hidden');
            document.getElementById('vote-screen').classList.remove('hidden');
            updateVoteScreen();
        }

        function startSuddenDeath() { startVoting(suddenDeathCandidateIndices); }

        function updateVoteScreen() {
            const currentVoter = alivePlayers[voterIndex];
            document.getElementById('voter-title').innerText = `🗳️ ${currentVoter.name} さんの投票`;
            document.getElementById('vote-instruction').classList.remove('hidden');
            document.getElementById('open-vote-btn').classList.remove('hidden');
            document.getElementById('vote-options-area').classList.add('hidden');
        }

        function openVoteOptions() {
            const currentVoter = alivePlayers[voterIndex];
            document.getElementById('vote-instruction').classList.add('hidden');
            document.getElementById('open-vote-btn').classList.add('hidden');
            document.getElementById('vote-options-area').classList.remove('hidden');

            const btnList = document.getElementById('vote-buttons-list');
            btnList.innerHTML = '';

            let candidates = suddenDeathCandidateIndices ? alivePlayers.filter(p => suddenDeathCandidateIndices.includes(p.originalIndex)) : alivePlayers;
            let selectableCandidates = candidates.filter(t => t.originalIndex !== currentVoter.originalIndex);
            if (selectableCandidates.length === 0) selectableCandidates = candidates;

            selectableCandidates.forEach(target => {
                const btn = document.createElement('button');
                btn.className = 'action-target';
                btn.innerText = `👤 ${target.name} に投票する`;
                btn.onclick = () => castVote(target.originalIndex);
                btnList.appendChild(btn);
            });
        }

        function castVote(targetOriginalIndex) {
            voteCounts[targetOriginalIndex]++;
            voterIndex++;
            if (voterIndex < alivePlayers.length) updateVoteScreen();
            else showVoteResults();
        }

        function checkAndApplyLoverSuicide(newsArray) {
            if (!gameRules.loverSuicide) return;
            
            let deadAdded = true;
            while (deadAdded) {
                deadAdded = false;
                const couples = playerData.filter(p => p.role === 'カップル 💑');
                if (couples.length === 2) {
                    const c1 = couples[0];
                    const c2 = couples[1];
                    if (!c1.alive && c2.alive) {
                        c2.alive = false;
                        newsArray.push(`💔 カップルの死を悲しみ、「${c2.name}」さんが後追い自殺してしまいました...`);
                        deadAdded = true;
                    } else if (!c2.alive && c1.alive) {
                        c1.alive = false;
                        newsArray.push(`💔 カップルの死を悲しみ、「${c1.name}」さんが後追い自殺してしまいました...`);
                        deadAdded = true;
                    }
                }
                const lovers = playerData.filter(p => p.isLover);
                if (lovers.length === 2) {
                    const l1 = lovers[0];
                    const l2 = lovers[1];
                    if (!l1.alive && l2.alive) {
                        l2.alive = false;
                        newsArray.push(`💘 恋人の死を悲しみ、「${l2.name}」さんが後追い自殺してしまいました...`);
                        deadAdded = true;
                    } else if (!l2.alive && l1.alive) {
                        l1.alive = false;
                        newsArray.push(`💘 恋人の死を悲しみ、「${l1.name}」さんが後追い自殺してしまいました...`);
                        deadAdded = true;
                    }
                }
            }
        }

        function showVoteResults() {
            document.getElementById('vote-screen').classList.add('hidden');
            document.getElementById('vote-result-screen').classList.remove('hidden');

            const candidateIndices = Object.keys(voteCounts).map(Number);
            const sortedResults = candidateIndices.map(idx => ({
                name: playerData[idx].name,
                originalIndex: idx,
                votes: voteCounts[idx]
            })).sort((a, b) => b.votes - a.votes);

            const summaryDiv = document.getElementById('vote-summary');
            summaryDiv.innerHTML = '';
            sortedResults.forEach(res => {
                summaryDiv.innerHTML += `<div style="margin: 6px 0; font-size: 16px;">・<b>${res.name}</b> ： ${res.votes} 票</div>`;
            });

            const maxVotes = sortedResults[0].votes;
            const topCandidates = sortedResults.filter(r => r.votes === maxVotes);

            const announceDiv = document.getElementById('execution-announcement');
            const extraNewsDiv = document.getElementById('execution-extra-news');
            const toNightBtn = document.getElementById('to-night-btn');
            const revoteBtn = document.getElementById('revote-btn');
            extraNewsDiv.innerText = '';

            if (topCandidates.length === 1) {
                const execIdx = topCandidates[0].originalIndex;
                playerData[execIdx].alive = false;
                lastExecutedPlayer = playerData[execIdx];
                suddenDeathCandidateIndices = null;

                announceDiv.innerText = `最多得票の「${playerData[execIdx].name}」さんが処刑されました。`;

                if (playerData[execIdx].role === 'てるてる ☀️') {
                    triggerTeruteruWin(playerData[execIdx]);
                    return;
                }

                let extraNews = [];
                if (playerData[execIdx].role === '黒猫 🐈‍⬛') {
                    const remainingAlive = playerData.map((p, i) => ({ ...p, idx: i })).filter(p => p.alive);
                    if (remainingAlive.length > 0) {
                        const victim = remainingAlive[Math.floor(Math.random() * remainingAlive.length)];
                        playerData[victim.idx].alive = false;
                        extraNews.push(`💥 黒猫の道連れにより、「${victim.name}」さんも死亡しました！`);
                        
                        if (playerData[victim.idx].role === 'てるてる ☀️') {
                            triggerTeruteruWin(playerData[victim.idx]);
                            return;
                        }
                    }
                }

                checkAndApplyLoverSuicide(extraNews);
                if (extraNews.length > 0) extraNewsDiv.innerHTML = extraNews.join('<br>');

                toNightBtn.classList.remove('hidden');
                revoteBtn.classList.add('hidden');
            } else {
                suddenDeathCandidateIndices = topCandidates.map(c => c.originalIndex);
                const names = topCandidates.map(c => c.name).join('、');
                announceDiv.innerText = `「${names}」が同票（${maxVotes}票）です！`;
                toNightBtn.classList.add('hidden');
                revoteBtn.classList.remove('hidden');
            }
        }

        function triggerTeruteruWin(teruteruPlayer) {
            if (timerInterval) clearInterval(timerInterval);
            document.getElementById('vote-result-screen').classList.add('hidden');
            document.getElementById('gameover-screen').classList.remove('hidden');

            document.getElementById('winner-title').innerText = `☀️ てるてる（第3陣営）の単独勝利！`;
            document.getElementById('winner-reason').innerText = `昼の投票により「${teruteruPlayer.name}」さんが処刑されたため、てるてるの単独勝利となりました！`;

            const finalList = document.getElementById('final-player-list');
            finalList.innerHTML = '';
            playerData.forEach(p => {
                const badge = p.alive ? '<span class="badge-alive">生存</span>' : '<span class="badge-dead">死亡</span>';
                const loverLabel = p.isLover ? ' <span style="color:#ff6b81; font-weight:bold;">+ 💘恋人</span>' : '';
                finalList.innerHTML += `
                    <li style="padding: 8px; border-bottom: 1px solid #444; display: flex; justify-content: space-between;">
                        <span><b>${p.name}</b> （${p.role}${loverLabel}）</span>
                        ${badge}
                    </li>`;
            });
        }

        function proceedToNight() {
            if (checkWinCondition()) return;

            playerData.forEach(p => {
                if (p.role === '罠師 🪤' && p.trapState === 'setting') {
                    p.trapState = 'active';
                }
            });

            alivePlayers = playerData.map((p, idx) => ({ ...p, originalIndex: idx })).filter(p => p.alive);
            nightActorIndex = 0;
            wolfVotes = {};
            nightVisits = [];
            foxKilledIndex = null;

            document.getElementById('vote-result-screen').classList.add('hidden');
            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('night-screen').classList.remove('hidden');
            updateNightScreen();
        }

        function updateNightScreen() {
            const actor = alivePlayers[nightActorIndex];
            document.getElementById('night-player-title').innerText = `🌙 ${actor.name} さんの夜の行動`;
            document.getElementById('night-instruction').classList.remove('hidden');
            document.getElementById('open-night-btn').classList.remove('hidden');
            document.getElementById('night-action-area').classList.add('hidden');
        }

        function openNightAction() {
            const actorWrapper = alivePlayers[nightActorIndex];
            const actor = playerData[actorWrapper.originalIndex];
            const actorOrigIdx = actorWrapper.originalIndex;

            document.getElementById('night-instruction').classList.add('hidden');
            document.getElementById('open-night-btn').classList.add('hidden');
            document.getElementById('night-action-area').classList.remove('hidden');

            const infoBox = document.getElementById('night-role-info');
            const targetsDiv = document.getElementById('night-targets-list');
            const doneBtn = document.getElementById('night-done-btn');

            targetsDiv.innerHTML = '';
            doneBtn.classList.add('hidden');

            let prefixNotice = '';

            if (gameRules.confusion && dayCount === 2 && actorOrigIdx === confusedPlayerIndex && !actor.confusionRevealed) {
                actor.confusionRevealed = true;
                prefixNotice += `🌀 <b>【混乱解除】</b> 昨夜まで隠されていたあなたの本当の役職は <b>${actor.role}</b> でした！<br><br>`;
            }

            let isDrunkActingFake = false;
            if (actor.role === '酔っぱらい 🍺') {
                if (dayCount >= 2 && !actor.drunkSobered) {
                    actor.drunkSobered = true;
                    prefixNotice += `🍺 <b>【酔いが覚めた！】</b> 頭がスッキリしました……！<br>実はこれまで思い込んでいた役職ではなく、あなたの本当の正体は <b>酔っぱらい（村人 🍺）</b> でした！<br><br>`;
                } else if (dayCount === 1 && !actor.drunkSobered) {
                    isDrunkActingFake = true;
                }
            }

            const actingRole = isDrunkActingFake ? actor.fakeRole : actor.role;

            if (actingRole === '人狼 🐺') {
                let noticeHtml = prefixNotice + '🐺 あなたは人狼です。今夜襲撃する人を1人選んでください。';
                
                const totalWolvesCount = playerData.filter(p => p.role === '人狼 🐺').length;
                if (totalWolvesCount >= 2) {
                    noticeHtml += `<br><br><span style="color: #ffa502; font-size: 13px;">※人狼が複数いる場合、投票数が同率の場合は最後の人狼が投票した人が殺されます。</span>`;
                }
                infoBox.innerHTML = noticeHtml;

                let targetMap = {};
                for (let wIndex in wolfVotes) {
                    let targetIdx = wolfVotes[wIndex];
                    let wolfObj = playerData[wIndex];
                    if (!targetMap[targetIdx]) targetMap[targetIdx] = [];
                    targetMap[targetIdx].push(wolfObj.name);
                }

                let aggregatedChoices = [];
                for (let tIdx in targetMap) {
                    aggregatedChoices.push({
                        targetIdx: Number(tIdx),
                        targetName: playerData[tIdx].name,
                        wolves: targetMap[tIdx]
                    });
                }

                if (aggregatedChoices.length === 0 || isDrunkActingFake) {
                    renderOtherTargetsList(actorOrigIdx, isDrunkActingFake, targetsDiv, []);
                } else {
                    let topChoiceBox = document.createElement('div');
                    topChoiceBox.style.marginBottom = '12px';
                    topChoiceBox.innerHTML = `<div style="font-size: 13px; color: #ff6b81; font-weight: bold; margin-bottom: 4px;">📜 前の人狼たちの選択肢：</div>`;

                    aggregatedChoices.forEach(ac => {
                        const btn = document.createElement('button');
                        btn.className = 'action-target';
                        btn.style.borderColor = '#ff6b81';
                        
                        let wolfNamesStr = ac.wolves.join('、');
                        let countStr = ac.wolves.length > 1 ? `（${ac.wolves.length}票）` : '';
                        btn.innerText = `🩸 ${wolfNamesStr} が選んだ： ${ac.targetName} を襲撃する ${countStr}`;
                        
                        btn.onclick = () => {
                            if (!isDrunkActingFake) {
                                wolfVotes[actorOrigIdx] = ac.targetIdx;
                            }
                            nextNightPlayer();
                        };
                        topChoiceBox.appendChild(btn);
                    });
                    targetsDiv.appendChild(topChoiceBox);

                    const alreadyChosenIndices = aggregatedChoices.map(ac => ac.targetIdx);
                    const remainingCount = alivePlayers.filter(t => t.role !== '人狼 🐺' && !alreadyChosenIndices.includes(t.originalIndex)).length;

                    if (remainingCount > 0) {
                        const otherBtn = document.createElement('button');
                        otherBtn.className = 'secondary';
                        otherBtn.innerText = '👤 ほかの人を選ぶ';
                        otherBtn.onclick = () => {
                            otherBtn.classList.add('hidden');
                            renderOtherTargetsList(actorOrigIdx, isDrunkActingFake, targetsDiv, alreadyChosenIndices);
                        };
                        targetsDiv.appendChild(otherBtn);
                    }
                }

            } else if (actingRole === '占い師 🔮') {
                infoBox.innerHTML = prefixNotice + '🔮 あなたは占い師です。今夜占う人を1人選んでください。';
                alivePlayers.forEach(t => {
                    if (t.originalIndex !== actorOrigIdx) {
                        const btn = document.createElement('button');
                        btn.className = 'action-target';
                        btn.innerText = `✨ ${t.name} を占う`;
                        btn.onclick = () => {
                            const targetPlayer = playerData[t.originalIndex];
                            if (isDrunkActingFake) {
                                const randomWolf = Math.random() < 0.5;
                                targetsDiv.innerHTML = '';
                                infoBox.innerHTML = `🔮 占い結果：<b>${targetPlayer.name}</b> さんは <span style="color:${randomWolf ? '#ff4757' : '#2ed573'}; font-size:18px;">【${randomWolf ? '人狼' : '人間'}】</span> です。`;
                                doneBtn.classList.remove('hidden');
                            } else {
                                nightVisits.push({
                                    visitorIndex: actorOrigIdx,
                                    targetIndex: t.originalIndex,
                                    type: 'seer'
                                });

                                if (targetPlayer.role === '罠師 🪤' && targetPlayer.trapState === 'active') {
                                    targetsDiv.innerHTML = '';
                                    infoBox.innerHTML = `✨ <b>${targetPlayer.name}</b> さんのもとへ占いに向かいました...`;
                                    doneBtn.classList.remove('hidden');
                                } else {
                                    const isWolf = targetPlayer.role === '人狼 🐺';
                                    if (targetPlayer.role === '狐 🦊') foxKilledIndex = t.originalIndex;
                                    targetsDiv.innerHTML = '';
                                    infoBox.innerHTML = `🔮 占い結果：<b>${targetPlayer.name}</b> さんは <span style="color:${isWolf ? '#ff4757' : '#2ed573'}; font-size:18px;">【${isWolf ? '人狼' : '人間'}】</span> です。`;
                                    doneBtn.classList.remove('hidden');
                                }
                            }
                        };
                        targetsDiv.appendChild(btn);
                    }
                });
            } else if (actingRole === '騎士 🛡️') {
                infoBox.innerHTML = prefixNotice + '🛡️ あなたは騎士です。今夜護衛する人を1人選んでください。';
                alivePlayers.forEach(t => {
                    if (!gameRules.selfGuard && t.originalIndex === actorOrigIdx) return;
                    if (!gameRules.continuousGuard && t.originalIndex === lastGuardedIndex) return;

                    const btn = document.createElement('button');
                    btn.className = 'action-target';
                    btn.innerText = `🛡️ ${t.name} を護衛する`;
                    btn.onclick = () => {
                        if (!isDrunkActingFake) {
                            nightVisits.push({
                                visitorIndex: actorOrigIdx,
                                targetIndex: t.originalIndex,
                                type: 'guard'
                            });
                            lastGuardedIndex = t.originalIndex;
                        }
                        nextNightPlayer();
                    };
                    targetsDiv.appendChild(btn);
                });
            } else if (actingRole === '罠師 🪤') {
                if (isDrunkActingFake || actor.trapState === 'unused') {
                    infoBox.innerHTML = prefixNotice + '🪤 あなたは罠師です。<br>今夜、自分の家に罠を設置しますか？（1ゲームに1度だけ使用でき、<b>設置した次の夜</b>に発動します）';
                    
                    const setBtn = document.createElement('button');
                    setBtn.className = 'action-target';
                    setBtn.style.borderColor = '#ffa502';
                    setBtn.innerText = '🪤 今夜、自宅に罠を設置する（次の夜に発動）';
                    setBtn.onclick = () => {
                        if (!isDrunkActingFake) {
                            actor.trapState = 'setting';
                        }
                        infoBox.innerHTML = prefixNotice + '✅ 自宅に罠を仕掛けました！次の夜に訪れた人を返り討みにします。';
                        targetsDiv.innerHTML = '';
                        doneBtn.classList.remove('hidden');
                    };
                    targetsDiv.appendChild(setBtn);

                    const skipBtn = document.createElement('button');
                    skipBtn.className = 'secondary';
                    skipBtn.innerText = '⏸️ 今夜は罠を仕掛けない';
                    skipBtn.onclick = () => {
                        nextNightPlayer();
                    };
                    targetsDiv.appendChild(skipBtn);

                } else if (actor.trapState === 'active') {
                    infoBox.innerHTML = prefixNotice + '🪤 昨夜設置した罠が<b>今夜発動中</b>です！<br>今夜あなたのもとを訪れた人を返り討ちにします。';
                    doneBtn.classList.remove('hidden');
                } else {
                    infoBox.innerHTML = prefixNotice + '🪤 あなたの罠はすでに使用済み（破壊済み）です。今夜は特別な行動はありません。';
                    doneBtn.classList.remove('hidden');
                }
            } else if (actingRole === '霊能者 👻') {
                if (isDrunkActingFake) {
                    const randResult = Math.random() < 0.5;
                    const execMsg = lastExecutedPlayer ? `昨日処刑された <b>${lastExecutedPlayer.name}</b> さんは <span style="color:${randResult ? '#ff4757' : '#2ed573'};">【${randResult ? '人狼' : '人間'}】</span> でした。` : 'まだ処刑された人はいません。';
                    infoBox.innerHTML = prefixNotice + `👻 霊能結果：<br>${execMsg}`;
                } else {
                    const execMsg = lastExecutedPlayer ? `昨日処刑された <b>${lastExecutedPlayer.name}</b> さんは <span style="color:${lastExecutedPlayer.role === '人狼 🐺' ? '#ff4757' : '#2ed573'};">【${lastExecutedPlayer.role === '人狼 🐺' ? '人狼' : '人間'}】</span> でした。` : 'まだ処刑された人はいません。';
                    infoBox.innerHTML = prefixNotice + `👻 霊能結果：<br>${execMsg}`;
                }
                doneBtn.classList.remove('hidden');
            } else {
                infoBox.innerHTML = prefixNotice + `あなたの役職（${actingRole}）は夜に特別な操作はありません。<br>下の3つのボタンから好きなものを1つ押してください：`;
                
                const funButtons = [
                    "🌙 今夜はぐっすり眠れそうです",
                    "👀 何だか物音がした気がする...",
                    "✨ 特になにも起きませんように"
                ];

                funButtons.forEach((text, i) => {
                    const btn = document.createElement('button');
                    btn.className = 'action-target';
                    btn.innerText = `🔘 ${text}`;
                    btn.onclick = () => {
                        const msgs = [
                            "💤 爆睡して体力を回復しました！",
                            "🦇 夜風が少し冷たく感じました。",
                            "🍀 平穏な夜のひとときを過ごしました。"
                        ];
                        infoBox.innerHTML = prefixNotice + `💡 <b>結果:</b> ${msgs[i]}`;
                        targetsDiv.innerHTML = '';
                        doneBtn.classList.remove('hidden');
                    };
                    targetsDiv.appendChild(btn);
                });
            }
        }

        function renderOtherTargetsList(actorOrigIdx, isDrunkActingFake, container, excludeIndices) {
            if (excludeIndices.length > 0) {
                const listTitle = document.createElement('div');
                listTitle.style.marginTop = '10px';
                listTitle.style.fontSize = '14px';
                listTitle.style.color = '#aaa';
                listTitle.innerText = '―― まだ選ばれていない参加者 ――';
                container.appendChild(listTitle);
            }

            alivePlayers.forEach(t => {
                const canSelect = isDrunkActingFake
                    ? (t.originalIndex !== actorOrigIdx)
                    : (t.role !== '人狼 🐺' && !excludeIndices.includes(t.originalIndex));

                if (canSelect) {
                    const btn = document.createElement('button');
                    btn.className = 'action-target';
                    btn.innerText = `🩸 ${t.name} を襲撃する`;
                    btn.onclick = () => {
                        if (!isDrunkActingFake) {
                            wolfVotes[actorOrigIdx] = t.originalIndex;
                        }
                        nextNightPlayer();
                    };
                    container.appendChild(btn);
                }
            });
        }

        function nextNightPlayer() {
            nightActorIndex++;
            if (nightActorIndex < alivePlayers.length) updateNightScreen();
            else resolveNightResults();
        }

        function resolveNightResults() {
            let morningNews = [];
            let deadNames = [];
            let trapDeadNames = []; // 罠によって死亡した人の名前リスト（役職非公開用）

            trapKillTriggeredByWolves = false;

            // 1. 人狼の襲撃ターゲット決定と、すべての投票した人狼の訪問を記録
            let targetOriginalIndex = null;
            let wolfIndices = Object.keys(wolfVotes).map(Number);

            if (wolfIndices.length > 0) {
                let counts = {};
                wolfIndices.forEach(wIdx => {
                    let tIdx = wolfVotes[wIdx];
                    counts[tIdx] = (counts[tIdx] || 0) + 1;
                });

                let maxCount = -1;
                let topTargets = [];
                for (let tIdx in counts) {
                    if (counts[tIdx] > maxCount) {
                        maxCount = counts[tIdx];
                        topTargets = [Number(tIdx)];
                    } else if (counts[tIdx] === maxCount) {
                        topTargets.push(Number(tIdx));
                    }
                }

                if (topTargets.length === 1) {
                    targetOriginalIndex = topTargets[0];
                } else {
                    let lastWolfIdx = wolfIndices[wolfIndices.length - 1];
                    targetOriginalIndex = wolfVotes[lastWolfIdx];
                }

                // 人狼全員がそのターゲットを襲撃するために訪れたとして記録
                wolfIndices.forEach(wIdx => {
                    if (wolfVotes[wIdx] === targetOriginalIndex) {
                        nightVisits.push({
                            visitorIndex: wIdx,
                            targetIndex: targetOriginalIndex,
                            type: 'wolf'
                        });
                    }
                });
            }

            // 2. 罠の判定処理
            let effectiveGuardTarget = null;
            let wolfAttackBlockedByTrap = false;

            // 罠師の家を訪れた訪問者を収集
            let trapTargetVisitedList = [];
            nightVisits.forEach(visit => {
                const target = playerData[visit.targetIndex];
                if (target.role === '罠師 🪤' && target.trapState === 'active') {
                    trapTargetVisitedList.push(visit);
                }
            });

            if (trapTargetVisitedList.length > 0) {
                const trapper = playerData.find(p => p.role === '罠師 🪤');
                // 罠師自身が生存している場合のみ発動
                if (trapper && trapper.alive) {
                    // 人狼が2人以上含まれているかチェック
                    let wolfVisitsToTrap = trapTargetVisitedList.filter(v => playerData[v.visitorIndex].role === '人狼 🐺' && playerData[v.visitorIndex].alive);

                    if (wolfVisitsToTrap.length >= 2) {
                        // 人狼が2人以上同時期に来た場合：ランダムで1人の人狼が罠によって死亡
                        let unluckyWolfVisit = wolfVisitsToTrap[Math.floor(Math.random() * wolfVisitsToTrap.length)];
                        let visitor = playerData[unluckyWolfVisit.visitorIndex];
                        visitor.alive = false;
                        trapDeadNames.push(visitor.name);

                        wolfAttackBlockedByTrap = true; // 襲撃自体は阻止または軽減される
                        trapKillTriggeredByWolves = true;
                    } else {
                        // 騎士や占い師、または人狼1人の場合：最初に来た1人が罠によって死亡
                        let firstVisit = trapTargetVisitedList[0];
                        let visitor = playerData[firstVisit.visitorIndex];
                        if (visitor.alive) {
                            visitor.alive = false;
                            trapDeadNames.push(visitor.name);

                            if (firstVisit.type === 'wolf') {
                                wolfAttackBlockedByTrap = true;
                                trapKillTriggeredByWolves = true;
                            }
                        }
                    }

                    // 罠は1度発動したら壊れる
                    trapper.trapState = 'used';
                }
            }

            // 誰も引っかからなかった場合でも、設置されていた罠はその夜が終わると壊れる
            playerData.forEach(p => {
                if (p.role === '罠師 🪤' && p.trapState === 'active') {
                    p.trapState = 'used';
                }
            });

            // 罠にかからなかった人たちの通常行動処理（護衛など）
            nightVisits.forEach(visit => {
                const visitor = playerData[visit.visitorIndex];
                if (!visitor.alive) return; // 罠で死んだ人は行動無効

                if (visit.type === 'guard') {
                    effectiveGuardTarget = visit.targetIndex;
                }
            });

            // 3. 通常の人狼襲撃結果の反映
            if (targetOriginalIndex !== null && !wolfAttackBlockedByTrap) {
                const target = playerData[targetOriginalIndex];
                if (targetOriginalIndex === effectiveGuardTarget) {
                    // 騎士の護衛成功
                } else if (target.role === '狐 🦊') {
                    // 狐は襲撃では死亡しない
                } else if (target.alive) {
                    target.alive = false;
                    deadNames.push(target.name);

                    if (target.role === '猫又 🐾') {
                        const aliveWolves = playerData.map((p, i) => ({ ...p, idx: i })).filter(p => p.alive && p.role === '人狼 🐺');
                        if (aliveWolves.length > 0) {
                            const deadWolf = aliveWolves[Math.floor(Math.random() * aliveWolves.length)];
                            playerData[deadWolf.idx].alive = false;
                            deadNames.push(deadWolf.name);
                        }
                    }
                }
            }

            // 4. 占い師による狐の呪殺
            if (foxKilledIndex !== null && playerData[foxKilledIndex].alive) {
                const seerVisit = nightVisits.find(v => v.type === 'seer' && v.targetIndex === foxKilledIndex);
                if (!seerVisit || playerData[seerVisit.visitorIndex].alive) {
                    playerData[foxKilledIndex].alive = false;
                    deadNames.push(playerData[foxKilledIndex].name);
                }
            }

            // 5. カップル・恋人の後追い自殺判定
            let suicideNames = [];
            if (gameRules.loverSuicide) {
                let checkAgain = true;
                while (checkAgain) {
                    checkAgain = false;
                    const couples = playerData.filter(p => p.role === 'カップル 💑');
                    if (couples.length === 2) {
                        const c1 = couples[0];
                        const c2 = couples[1];
                        if (!c1.alive && c2.alive && !deadNames.includes(c2.name) && !trapDeadNames.includes(c2.name) && !suicideNames.includes(c2.name)) {
                            c2.alive = false;
                            suicideNames.push(c2.name);
                            checkAgain = true;
                        } else if (!c2.alive && c1.alive && !deadNames.includes(c1.name) && !trapDeadNames.includes(c1.name) && !suicideNames.includes(c1.name)) {
                            c1.alive = false;
                            suicideNames.push(c1.name);
                            checkAgain = true;
                        }
                    }
                    const lovers = playerData.filter(p => p.isLover);
                    if (lovers.length === 2) {
                        const l1 = lovers[0];
                        const l2 = lovers[1];
                        if (!l1.alive && l2.alive && !deadNames.includes(l2.name) && !trapDeadNames.includes(l2.name) && !suicideNames.includes(l2.name)) {
                            l2.alive = false;
                            suicideNames.push(l2.name);
                            checkAgain = true;
                        } else if (!l2.alive && l1.alive && !deadNames.includes(l1.name) && !trapDeadNames.includes(l1.name) && !suicideNames.includes(l1.name)) {
                            l1.alive = false;
                            suicideNames.push(l1.name);
                            checkAgain = true;
                        }
                    }
                }
            }

            // 6. 朝のニュースメッセージ組み立て（罠死は役職非公開）
            let allDeadAnnouncements = [];
            trapDeadNames.forEach(name => {
                allDeadAnnouncements.push(`<b>${name}</b> さん（罠によって死亡・役職不明）`);
            });
            deadNames.forEach(name => {
                if (!allDeadAnnouncements.some(item => item.includes(name))) {
                    allDeadAnnouncements.push(`<b>${name}</b> さん`);
                }
            });
            suicideNames.forEach(name => {
                allDeadAnnouncements.push(`<b>${name}</b> さん`);
            });

            if (allDeadAnnouncements.length === 0) {
                morningNews.push('☀️ 朝になりました。昨晩の犠牲者はいませんでした！');
            } else {
                morningNews.push(`☀️ 朝になりました。昨晩の犠牲者は ${allDeadAnnouncements.join('、')} でした...`);
            }

            dayCount++;
            enterDayPhase(morningNews);
        }

        function checkWinCondition() {
            const aliveWolves = playerData.filter(p => p.alive && p.role === '人狼 🐺').length;
            const initialWolvesCount = playerData.filter(p => p.role === '人狼 🐺').length;
            const aliveFoxes = playerData.filter(p => p.alive && p.role === '狐 🦊').length;
            const aliveHumans = playerData.filter(p => p.alive && p.role !== '人狼 🐺' && p.role !== '狐 🦊').length;

            const lovers = playerData.filter(p => p.isLover);
            const bothLoversAlive = (lovers.length === 2 && lovers[0].alive && lovers[1].alive);
            const loversAreBothHuman = (lovers.length === 2 && isHumanSide(lovers[0].role) && isHumanSide(lovers[1].role));
            const loversAreBothWolf = (lovers.length === 2 && lovers[0].role === '人狼 🐺' && lovers[1].role === '人狼 🐺');
            const loversAreThirdParty = (lovers.length === 2 && !loversAreBothHuman && !loversAreBothWolf);

            let winner = null;
            let reason = '';

            const humanWinCondition = (aliveWolves === 0);

            if (humanWinCondition) {
                if (loversAreThirdParty && bothLoversAlive) {
                    winner = '💘 恋人（第3陣営）の単独勝利！';
                    reason = `人狼が全滅したとき、愛し合う2人（${lovers[0].name}・${lovers[1].name}）が生き残っていました！`;
                } else if (aliveFoxes > 0) {
                    winner = '🦊 狐（第3陣営）の単独勝利！';
                    reason = '人狼が全滅したとき、狐が生き残っていました！';
                } else {
                    winner = '🎉 人間陣営の勝利！';
                    if (trapKillTriggeredByWolves) {
                        reason = '罠師が仕掛けた罠によって人狼たちが返り討ちに遭い、全滅しました！';
                    } else {
                        reason = (initialWolvesCount >= 2) ? '人狼をすべて（全員）追放しました！' : '村に潜んでいた人狼を追放しました！';
                    }
                    if (loversAreBothHuman && bothLoversAlive) {
                        reason += `\n💘 人間陣営の恋人（${lovers[0].name}・${lovers[1].name}）も共に勝利しました！`;
                    }
                }
            } else if (aliveWolves >= aliveHumans) {
                if (loversAreThirdParty && bothLoversAlive) {
                    winner = '💘 恋人（第3陣営）の単独勝利！';
                    reason = `人狼が村を支配したとき、異なる陣営の恋人（${lovers[0].name}・${lovers[1].name}）が2人とも生き残っていました！`;
                } else if (aliveFoxes > 0) {
                    winner = '🦊 狐（第3陣営）の単独勝利！';
                    reason = '人狼が村を支配したとき、狐が生き残っていました！';
                } else {
                    winner = '🐺 人狼陣営の勝利！';
                    reason = '人狼の数が人間と同数以上になり、村は人狼に支配されました...';
                }
            }

            if (winner) {
                if (timerInterval) clearInterval(timerInterval);
                document.getElementById('game-screen').classList.add('hidden');
                document.getElementById('vote-result-screen').classList.add('hidden');
                document.getElementById('night-screen').classList.add('hidden');
                document.getElementById('gameover-screen').classList.remove('hidden');

                document.getElementById('winner-title').innerText = winner;
                document.getElementById('winner-reason').innerText = reason;

                const finalList = document.getElementById('final-player-list');
                finalList.innerHTML = '';
                playerData.forEach(p => {
                    const badge = p.alive ? '<span class="badge-alive">生存</span>' : '<span class="badge-dead">死亡</span>';
                    const loverLabel = p.isLover ? ' <span style="color:#ff6b81; font-weight:bold;">+ 💘恋人</span>' : '';
                    const drunkLabel = (p.role === '酔っぱらい 🍺') ? `（思い込み：${p.fakeRole}）` : '';
                    finalList.innerHTML += `
                        <li style="padding: 8px; border-bottom: 1px solid #444; display: flex; justify-content: space-between;">
                            <span><b>${p.name}</b> （${p.role}${drunkLabel}${loverLabel}）</span>
                            ${badge}
                        </li>`;
                });
                return true;
            }
            return false;
        }
    </script>
</body>
</html>
