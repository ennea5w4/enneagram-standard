<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>エニアグラム診断 - スタンダード版</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary-color: #4a90e2;
            --bg-color: #f5f7fa;
            --card-bg: #ffffff;
            --text-color: #333;
        }

        body {
            font-family: 'Helvetica Neue', Arial, 'Hiragino Kaku Gothic ProN', 'Hiragino Sans', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            width: 90%;
            max-width: 600px;
            background: var(--card-bg);
            padding: 2rem;
            border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
        }

        .progress-container {
            width: 100%;
            height: 8px;
            background: #eee;
            border-radius: 4px;
            margin-bottom: 2rem;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            background: var(--primary-color);
            width: 0%;
            transition: width 0.3s ease;
        }

        .question-box { margin-bottom: 2rem; }
        .question-text { font-size: 1.2rem; font-weight: bold; margin-bottom: 1.5rem; }

        .options { display: grid; gap: 10px; }
        .btn {
            padding: 12px;
            border: 2px solid var(--primary-color);
            background: transparent;
            color: var(--primary-color);
            border-radius: 8px;
            cursor: pointer;
            font-size: 1rem;
            transition: all 0.2s;
        }
        .btn:hover {
            background: var(--primary-color);
            color: white;
        }

        #result-area { display: none; }
        canvas { margin-top: 1.5rem; }
        
        .hidden { display: none; }
    </style>
</head>
<body>

<div class="container">
    <div id="quiz-area">
        <div class="progress-container">
            <div id="progress-bar" class="progress-bar"></div>
        </div>
        <div class="question-box">
            <p id="question-number" style="color: #888;">Question 1</p>
            <p id="question-text" class="question-text">読み込み中...</p>
        </div>
        <div class="options">
            <button class="btn" onclick="answer(3)">非常に当てはまる</button>
            <button class="btn" onclick="answer(2)">やや当てはまる</button>
            <button class="btn" onclick="answer(1)">あまり当てはまらない</button>
            <button class="btn" onclick="answer(0)">全く当てはまらない</button>
        </div>
    </div>

    <div id="result-area">
        <h2>あなたの診断結果</h2>
        <h1 id="top-type" style="color: var(--primary-color);">タイプX</h1>
        <p id="type-description">タイプの説明がここに表示されます。</p>
        <canvas id="typeChart"></canvas>
        <button class="btn" style="margin-top: 2rem;" onclick="location.reload()">もう一度診断する</button>
    </div>
</div>

<script>
    // 質問リスト（各タイプ2問ずつの簡易版例：実際には各5〜10問推奨）
    const questions = [
        { text: "完璧主義で、物事が正しく行われないと気になる。", type: 1 },
        { text: "他人のニーズに敏感で、助けたいと思う。", type: 2 },
        { text: "成功することや、効率的に成果を出すことが重要だ。", type: 3 },
        { text: "自分は他人とはどこか違い、独特であると感じる。", type: 4 },
        { text: "情報を収集し、物事を客観的に分析するのが好きだ。", type: 5 },
        { text: "周囲の安全や、ルールを守ることに慎重になる。", type: 6 },
        { text: "常に新しい楽しい体験を求め、退屈を避けたい。", type: 7 },
        { text: "強さを重視し、物事をコントロールしていたい。", type: 8 },
        { text: "平和を好み、対立を避けて穏やかに過ごしたい。", type: 9 }
    ];

    let currentIdx = 0;
    let scores = { 1:0, 2:0, 3:0, 4:0, 5:0, 6:0, 7:0, 8:0, 9:0 };

    const typeDetails = {
        1: "【改革する人】常に正しく、改善しようとする努力家です。",
        2: "【助ける人】温厚で、他者の助けになることに喜びを感じます。",
        3: "【達成する人】目標に向かって効率的に行動し、成功を収めます。",
        4: "【個性を求める人】感受性が豊かで、自分らしさを大切にします。",
        5: "【調べる人】観察力が鋭く、知識を深めることを好みます。",
        6: "【忠実な人】責任感が強く、慎重にリスクを回避します。",
        7: "【熱中する人】楽天的で好奇心旺盛、人生を楽しみます。",
        8: "【挑戦する人】自信に溢れ、リーダーシップを発揮します。",
        9: "【平和をもたらす人】穏やかで包容力があり、和を保ちます。"
    };

    function showQuestion() {
        if (currentIdx < questions.length) {
            document.getElementById('question-number').innerText = `Question ${currentIdx + 1} / ${questions.length}`;
            document.getElementById('question-text').innerText = questions[currentIdx].text;
            const progress = (currentIdx / questions.length) * 100;
            document.getElementById('progress-bar').style.width = `${progress}%`;
        } else {
            showResult();
        }
    }

    function answer(point) {
        const type = questions[currentIdx].type;
        scores[type] += point;
        currentIdx++;
        showQuestion();
    }

    function showResult() {
        document.getElementById('quiz-area').classList.add('hidden');
        document.getElementById('result-area').style.display = 'block';

        // 最高得点のタイプを特定
        let topType = Object.keys(scores).reduce((a, b) => scores[a] > scores[b] ? a : b);
        document.getElementById('top-type').innerText = `タイプ ${topType}`;
        document.getElementById('type-description').innerText = typeDetails[topType];

        // チャート描画
        const ctx = document.getElementById('typeChart').getContext('2d');
        new Chart(ctx, {
            type: 'radar',
            data: {
                labels: ['Type 1', 'Type 2', 'Type 3', 'Type 4', 'Type 5', 'Type 6', 'Type 7', 'Type 8', 'Type 9'],
                datasets: [{
                    label: 'あなたのスコア',
                    data: Object.values(scores),
                    backgroundColor: 'rgba(74, 144, 226, 0.2)',
                    borderColor: 'rgba(74, 144, 226, 1)',
                    borderWidth: 2
                }]
            },
            options: {
                scales: {
                    r: { beginAtZero: true, max: 10 } // 質問数に応じて調整
                }
            }
        });
    }

    showQuestion();
</script>

</body>
</html># enneagram-standard
