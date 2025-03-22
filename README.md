# fj
Desenvolvimento Pessoal Força dos Jovens
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Desenvolvimento Pessoal - Login</title>
    <link rel="stylesheet" href="styles.css">
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-firestore.js"></script>
</head>
<body>
    <div class="container">
        <h1>Login - Desenvolvimento Pessoal</h1>
        <form id="loginForm">
            <input type="email" id="email" placeholder="Digite seu e-mail" required>
            <input type="password" id="password" placeholder="Digite sua senha" required>
            <select id="tipoUsuario" required>
                <option value="">Selecione seu perfil</option>
                <option value="jovem">Jovem</option>
                <option value="mestre">Mestre</option>
            </select>
            <button type="submit">Entrar</button>
        </form>
        <button id="signup">Criar Conta</button>
    </div>

    <script type="module">
        // Configuração Firebase
        const firebaseConfig = {
            apiKey: "SUA_API_KEY",
            authDomain: "SEU_PROJETO.firebaseapp.com",
            projectId: "SEU_PROJETO",
            storageBucket: "SEU_PROJETO.appspot.com",
            messagingSenderId: "SENDER_ID",
            appId: "APP_ID"
        };

        const app = firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        document.getElementById('loginForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            const tipoUsuario = document.getElementById('tipoUsuario').value;

            auth.signInWithEmailAndPassword(email, password)
                .then((userCredential) => {
                    const user = userCredential.user;
                    if (tipoUsuario === "jovem") {
                        window.location.href = `jovem_dashboard.html?user=${user.uid}`;
                    } else if (tipoUsuario === "mestre") {
                        window.location.href = `mestre_dashboard.html?user=${user.uid}`;
                    }
                })
                .catch((error) => alert("Erro ao fazer login: " + error.message));
        });

        document.getElementById('signup').addEventListener('click', () => {
            const email = prompt('Digite seu e-mail:');
            const password = prompt('Digite sua senha:');
            auth.createUserWithEmailAndPassword(email, password)
                .then(() => alert('Conta criada com sucesso!'))
                .catch((error) => alert("Erro ao criar conta: " + error.message));
        });
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel do Jovem</title>
    <link rel="stylesheet" href="styles.css">
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-firestore.js"></script>
</head>
<body>
    <h1>Painel do Jovem</h1>
    <h2 id="userName"></h2>
    <form id="dailyForm">
        <label>Física:</label> <input type="number" id="fisica" min="1" max="5" required><br>
        <label>Espiritual:</label> <input type="number" id="espiritual" min="1" max="5" required><br>
        <label>Intelectual:</label> <input type="number" id="intelectual" min="1" max="5" required><br>
        <label>Social:</label> <input type="number" id="social" min="1" max="5" required><br>
        <button type="submit">Salvar Pontuação</button>
    </form>

    <h3>Resumo da Semana</h3>
    <div id="weeklySummary">Carregando...</div>

    <script type="module">
        const firebaseConfig = {
            apiKey: "SUA_API_KEY",
            authDomain: "SEU_PROJETO.firebaseapp.com",
            projectId: "SEU_PROJETO",
            storageBucket: "SEU_PROJETO.appspot.com",
            messagingSenderId: "SENDER_ID",
            appId: "APP_ID"
        };

        const app = firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        const userId = new URLSearchParams(window.location.search).get('user');
        const userName = document.getElementById('userName');

        db.collection("users").doc(userId).get().then(doc => {
            userName.textContent = `Olá, ${doc.data().nome}!`;
        });

        document.getElementById('dailyForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const data = {
                fisica: parseInt(document.getElementById('fisica').value),
                espiritual: parseInt(document.getElementById('espiritual').value),
                intelectual: parseInt(document.getElementById('intelectual').value),
                social: parseInt(document.getElementById('social').value),
                date: new Date().toISOString().split('T')[0]
            };

            db.collection('users').doc(userId).collection('entries').add(data)
                .then(() => alert('Pontuação salva!'))
                .catch(err => alert('Erro ao salvar: ' + err.message));
        });

        const loadWeeklySummary = () => {
            const today = new Date();
            const weekAgo = new Date();
            weekAgo.setDate(today.getDate() - 7);

            db.collection('users').doc(userId).collection('entries')
                .where('date', '>=', weekAgo.toISOString().split('T')[0])
                .get().then(snapshot => {
                    let total = 0;
                    let count = 0;

                    snapshot.forEach(doc => {
                        const data = doc.data();
                        total += data.fisica + data.espiritual + data.intelectual + data.social;
                        count++;
                    });

                    const avg = count ? (total / (count * 4)).toFixed(2) : 0;
                    const status = avg >= 4 ? "☀️ Excelente!" : avg >= 2.5 ? "🌤️ Bom, mas dá pra melhorar!" : "🌧️ Precisa focar mais!";
                    document.getElementById('weeklySummary').innerHTML = `Média da semana: ${avg} ⭐<br>Status: ${status}`;
                });
        };

        loadWeeklySummary();
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel do Mestre</title>
    <link rel="stylesheet" href="styles.css">
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-firestore.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Painel do Mestre</h1>
    <h2>Lista de Jovens</h2>
    <table border="1">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Média Semanal</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody id="jovensTable">Carregando...</tbody>
    </table>

    <h2>Gráfico de Desempenho</h2>
    <canvas id="graficoDesempenho" width="400" height="200"></canvas>

    <h2>Áreas Individuais</h2>
    <div id="areas">
        <button style="background-color: #4A90E2; color: #fff;" onclick="mostrarArea('espiritual')">Espiritual</button>
        <button style="background-color: #7ED321; color: #fff;" onclick="mostrarArea('social')">Social</button>
        <button style="background-color: #D0021B; color: #fff;" onclick="mostrarArea('fisica')">Físico</button>
        <button style="background-color: #F5A623; color: #fff;" onclick="mostrarArea('intelectual')">Intelectual</button>
    </div>
    <div id="conteudoArea">Selecione uma área para ver detalhes.</div>

    <script type="module">
        const firebaseConfig = {
            apiKey: "SUA_API_KEY",
            authDomain: "SEU_PROJETO.firebaseapp.com",
            projectId: "SEU_PROJETO",
            storageBucket: "SEU_PROJETO.appspot.com",
            messagingSenderId: "SENDER_ID",
            appId: "APP_ID"
        };

        const app = firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        const carregarJovens = async () => {
            const tableBody = document.getElementById('jovensTable');
            tableBody.innerHTML = '';
            let totaisPorArea = { fisica: 0, espiritual: 0, social: 0, intelectual: 0 };
            let count = 0;

            const usersSnapshot = await db.collection('users').get();

            usersSnapshot.forEach(async (doc) => {
                const userData = doc.data();
                const userId = doc.id;
                let total = 0;

                const entriesSnapshot = await db.collection('users').doc(userId).collection('entries').get();
                entriesSnapshot.forEach(entryDoc => {
                    const data = entryDoc.data();
                    totaisPorArea.fisica += data.fisica;
                    totaisPorArea.espiritual += data.espiritual;
                    totaisPorArea.intelectual += data.intelectual;
                    totaisPorArea.social += data.social;
                    total += data.fisica + data.espiritual + data.intelectual + data.social;
                    count++;
                });

                const avg = count ? (total / (count * 4)).toFixed(2) : 0;
                const status = avg >= 4 ? "☀️ Excelente!" : avg >= 2.5 ? "🌤️ Bom, mas dá pra melhorar!" : "🌧️ Precisa melhorar!";

                const row = `<tr>
                                <td>${userData.nome}</td>
                                <td>${avg}</td>
                                <td>${status}</td>
                            </tr>`;
                tableBody.innerHTML += row;
            });

            const ctx = document.getElementById('graficoDesempenho').getContext('2d');
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Física', 'Espiritual', 'Intelectual', 'Social'],
                    datasets: [{
                        label: 'Desempenho por Área',
                        data: [
                            totaisPorArea.fisica / count,
                            totaisPorArea.espiritual / count,
                            totaisPorArea.intelectual / count,
                            totaisPorArea.social / count
                        ],
                        backgroundColor: ['#D0021B', '#4A90E2', '#F5A623', '#7ED321'],
                    }]
                }
            });
        };

        const mostrarArea = (area) => {
            const conteudo = {
                espiritual: 'Dedicação ao desenvolvimento espiritual, meditações e reflexões pessoais.',
                social: 'Interações sociais, apoio aos amigos e participação em atividades coletivas.',
                fisica: 'Exercícios físicos, saúde e cuidados com o corpo.',
                intelectual: 'Estudo, aprendizado de novas habilidades e desenvolvimento mental.'
            };
            document.getElementById('conteudoArea').innerHTML = conteudo[area];
        };

        carregarJovens();
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel do Mestre</title>
    <link rel="stylesheet" href="styles.css">
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-firestore.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Painel do Mestre</h1>
    <h2>Lista de Jovens</h2>
    <table border="1">
        <thead>
            <tr>
                <th>Nome</th>
                <th>Média Semanal</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody id="jovensTable">Carregando...</tbody>
    </table>

    <h2>Gráfico de Desempenho</h2>
    <canvas id="graficoDesempenho" width="400" height="200"></canvas>

    <h2>Áreas Individuais</h2>
    <div id="areas">
        <button style="background-color: #4A90E2; color: #fff;" onclick="mostrarArea('espiritual')">Espiritual</button>
        <button style="background-color: #7ED321; color: #fff;" onclick="mostrarArea('social')">Social</button>
        <button style="background-color: #D0021B; color: #fff;" onclick="mostrarArea('fisica')">Físico</button>
        <button style="background-color: #F5A623; color: #fff;" onclick="mostrarArea('intelectual')">Intelectual</button>
    </div>
    <div id="conteudoArea">Selecione uma área para ver detalhes.</div>

    <script type="module">
        const firebaseConfig = {
            apiKey: "SUA_API_KEY",
            authDomain: "SEU_PROJETO.firebaseapp.com",
            projectId: "SEU_PROJETO",
            storageBucket: "SEU_PROJETO.appspot.com",
            messagingSenderId: "SENDER_ID",
            appId: "APP_ID"
        };

        const app = firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        const carregarJovens = async () => {
            const tableBody = document.getElementById('jovensTable');
            tableBody.innerHTML = '';
            let totaisPorArea = { fisica: 0, espiritual: 0, social: 0, intelectual: 0 };
            let count = 0;

            const usersSnapshot = await db.collection('users').get();

            usersSnapshot.forEach(async (doc) => {
                const userData = doc.data();
                const userId = doc.id;
                let total = 0;

                const entriesSnapshot = await db.collection('users').doc(userId).collection('entries').get();
                entriesSnapshot.forEach(entryDoc => {
                    const data = entryDoc.data();
                    totaisPorArea.fisica += data.fisica;
                    totaisPorArea.espiritual += data.espiritual;
                    totaisPorArea.intelectual += data.intelectual;
                    totaisPorArea.social += data.social;
                    total += data.fisica + data.espiritual + data.intelectual + data.social;
                    count++;
                });

                const avg = count ? (total / (count * 4)).toFixed(2) : 0;
                const status = avg >= 4 ? "☀️ Excelente!" : avg >= 2.5 ? "🌤️ Bom, mas dá pra melhorar!" : "🌧️ Precisa melhorar!";

                const row = `<tr>
                                <td>${userData.nome}</td>
                                <td>${avg}</td>
                                <td>${status}</td>
                            </tr>`;
                tableBody.innerHTML += row;
            });

            const ctx = document.getElementById('graficoDesempenho').getContext('2d');
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Física', 'Espiritual', 'Intelectual', 'Social'],
                    datasets: [{
                        label: 'Desempenho por Área',
                        data: [
                            totaisPorArea.fisica / count,
                            totaisPorArea.espiritual / count,
                            totaisPorArea.intelectual / count,
                            totaisPorArea.social / count
                        ],
                        backgroundColor: ['#D0021B', '#4A90E2', '#F5A623', '#7ED321'],
                    }]
                }
            });
        };

        const mostrarArea = (area) => {
            const conteudo = {
                espiritual: 'Dedicação ao desenvolvimento espiritual, meditações e reflexões pessoais.',
                social: 'Interações sociais, apoio aos amigos e participação em atividades coletivas.',
                fisica: 'Exercícios físicos, saúde e cuidados com o corpo.',
                intelectual: 'Estudo, aprendizado de novas habilidades e desenvolvimento mental.'
            };
            document.getElementById('conteudoArea').innerHTML = conteudo[area];
        };

        carregarJovens();
    </script>
</body>
</html>
