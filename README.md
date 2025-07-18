<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Quiz da Roleta</title>
  <style>
    /* teu CSS original aqui, sem mexer */
    body {
      font-family: Arial, sans-serif;
      background-color: #f0f0f0;
      margin: 0;
      padding: 0;
      text-align: center;
    }
    .container {
      margin: 50px auto;
      background: rgba(255, 255, 255, 0.95);
      padding: 20px;
      border-radius: 15px;
      display: inline-block;
      box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
      color: #000;
      max-width: 400px;
    }
    #roleta-container {
      position: relative;
      width: 200px;
      height: 200px;
      margin: 20px auto 30px auto;
    }
    #roleta {
      width: 200px;
      height: 200px;
      border-radius: 50%;
      background: conic-gradient(
        red 0% 10%, green 10% 20%, blue 20% 30%, yellow 30% 40%,
        orange 40% 50%, purple 50% 60%, pink 60% 70%, brown 70% 80%,
        cyan 80% 90%, gray 90% 100%
      );
      transition: transform 3s ease-out;
    }
    #seta {
      position: absolute;
      top: -25px;
      left: 50%;
      transform: translateX(-50%);
      width: 0;
      height: 0;
      border-left: 15px solid transparent;
      border-right: 15px solid transparent;
      border-top: 25px solid black;
      z-index: 10;
    }
    button {
      padding: 15px 30px;
      font-size: 20px;
      background-color: #4caf50;
      color: white;
      border: none;
      cursor: pointer;
      border-radius: 5px;
      margin: 10px;
    }
    button:hover:enabled {
      background-color: #45a049;
    }
    button:disabled {
      background-color: #888;
      cursor: not-allowed;
    }
    input {
      padding: 10px;
      margin: 10px;
      font-size: 16px;
      border-radius: 5px;
      border: 1px solid #ccc;
      width: calc(100% - 24px);
      box-sizing: border-box;
    }
    #resposta {
      margin: 20px 0;
      padding: 10px;
      font-weight: bold;
      min-height: 30px;
      border-radius: 5px;
    }
    #pontuacao {
      margin: 20px 0;
      font-size: 18px;
    }
    #classificacao-container {
      max-width: 600px;
      margin: 20px auto;
      background: white;
      border-radius: 15px;
      box-shadow: 0 0 10px rgba(0,0,0,0.2);
      padding: 20px;
      display: none;
      text-align: left;
    }
    #classificacao-container h2 {
      text-align: center;
      margin-bottom: 15px;
    }
    #tabela-classificacao {
      width: 100%;
      border-collapse: collapse;
    }
    #tabela-classificacao th, #tabela-classificacao td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: center;
    }
    #tabela-classificacao th {
      background-color: #4caf50;
      color: white;
    }
    #btn-voltar-jogo {
      background-color: #2196F3;
      margin-top: 15px;
      display: block;
      width: 100%;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <div class="container" id="jogo-container">
    <h1>Quiz da Roleta</h1>

    <div id="dados-jogador-div">
      <input type="text" id="nome" placeholder="Nome" />
      <input type="text" id="turma" placeholder="Turma" />
      <input type="text" id="ano" placeholder="Ano" />
    </div>
    <button id="salvar-dados">Salvar Dados e Iniciar</button>

    <div id="roleta-container" style="display:none;">
      <div id="seta"></div>
      <div id="roleta"></div>
    </div>

    <div id="pergunta-texto" style="font-weight:bold; font-size: 18px; min-height: 50px; margin: 20px 0;"></div>

    <div id="resposta"></div>

    <div class="botoes-resposta" style="display:none;">
      <button id="btn-verdadeiro">Verdadeiro</button>
      <button id="btn-falso">Falso</button>
    </div>

    <div id="pontuacao">Acertos: 0 | Erros: 0</div>

    <button id="btn-classificacao" style="display:none;">Ver Classificação</button>
  </div>

  <!-- Container para a classificação -->
  <div id="classificacao-container">
    <h2>Classificação</h2>
    <table id="tabela-classificacao">
      <thead>
        <tr>
          <th>Nome</th>
          <th>Turma</th>
          <th>Ano</th>
          <th>Acertos</th>
          <th>Erros</th>
        </tr>
      </thead>
      <tbody id="tabela-jogadores">
        <!-- Jogadores vão aparecer aqui -->
      </tbody>
    </table>
    <button id="btn-voltar-jogo">Voltar ao Jogo</button>
  </div>

  <script>
    document.addEventListener("DOMContentLoaded", function () {
      const perguntas = [
        { texto: "O processador (CPU) é considerado o cérebro do computador?", resposta: "Verdadeiro" },
        { texto: "A memória RAM armazena dados de forma permanente?", resposta: "Falso" },
        { texto: "O sistema operacional é um tipo de hardware?", resposta: "Falso" },
        { texto: "Um pendrive é um exemplo de dispositivo de armazenamento externo?", resposta: "Verdadeiro" },
        { texto: "Os navegadores de internet são usados para acessar sites na web?", resposta: "Verdadeiro" },
        { texto: "O teclado e o mouse são considerados dispositivos de saída?", resposta: "Falso" },
        { texto: "Um vírus de computador pode danificar arquivos e roubar informações?", resposta: "Verdadeiro" },
        { texto: "Wi-Fi é um tipo de conexão com fio?", resposta: "Falso" },
        { texto: "USB significa 'Unidade de Sistema Básico'?", resposta: "Falso" },
        { texto: "O software é a parte física do computador?", resposta: "Falso" }
      ];

      let perguntasRestantes = [];
      let perguntaAtual = null;
      let acertos = 0;
      let erros = 0;
      let perguntasRespondidas = 0;
      let jogador = {};

      const perguntaTexto = document.getElementById('pergunta-texto');
      const respostaTexto = document.getElementById('resposta');
      const pontuacao = document.getElementById('pontuacao');
      const botoesResposta = document.querySelector('.botoes-resposta');
      const salvarDadosBtn = document.getElementById('salvar-dados');
      const classificacaoBtn = document.getElementById('btn-classificacao');
      const roletaContainer = document.getElementById('roleta-container');
      const dadosJogadorDiv = document.getElementById('dados-jogador-div');
      const jogoContainer = document.getElementById('jogo-container');

      const classificacaoContainer = document.getElementById('classificacao-container');
      const tabelaJogadores = document.getElementById('tabela-jogadores');
      const btnVoltarJogo = document.getElementById('btn-voltar-jogo');

      // Usa o teu link do App Script aqui
      const SHEET_ENDPOINT = "https://script.google.com/macros/s/AKfycbyK4fjvS1ZKO_uWwLl4m99SozL6IZi3mbw_lHKiQm2o-_LzwiRtdpBUt3lrxwcuKH-4OA/exec";

      function atualizarPontuacao() {
        pontuacao.textContent = `Acertos: ${acertos} | Erros: ${erros}`;
      }
function fimDeJogo() {
  perguntaTexto.textContent = "Fim do jogo! Obrigado por jogar!";
  botoesResposta.style.display = 'none';
  roletaContainer.style.display = 'none';
  classificacaoBtn.style.display = 'inline-block';

  // limpa o texto da resposta
  respostaTexto.textContent = "";
  respostaTexto.style.backgroundColor = "";

  // Salvar jogador no localStorage
  let jogadores = JSON.parse(localStorage.getItem("jogadores")) || [];
  jogadores.push(jogador);
  localStorage.setItem("jogadores", JSON.stringify(jogadores));

  // Enviar dados para Google Sheets
  fetch(SHEET_ENDPOINT, {
    method: "POST",
    mode: "no-cors",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(jogador)
  }).catch(err => {
    console.log("Erro ao enviar dados para Google Sheets:", err);
  });
}

      function selecionarPerguntaAleatoria() {
        if (perguntasRestantes.length === 0 || perguntasRespondidas >= 10) {
          fimDeJogo();
          return;
        }
        const indice = Math.floor(Math.random() * perguntasRestantes.length);
        perguntaAtual = perguntasRestantes.splice(indice, 1)[0];
        perguntaTexto.textContent = perguntaAtual.texto;
        respostaTexto.textContent = "";
        respostaTexto.style.backgroundColor = "";
        botoesResposta.style.display = 'block';
      }

      function girarRoleta(callback) {
        const roleta = document.getElementById('roleta');

        roleta.style.transition = 'none';
        roleta.style.transform = 'rotate(0deg)';
        roleta.offsetHeight; // força aplicação do estilo

        const randomDegree = Math.floor(Math.random() * 360) + 3600; // 10 voltas + aleatório
        roleta.style.transition = 'transform 3s ease-out';
        roleta.style.transform = `rotate(${randomDegree}deg)`;

        setTimeout(() => {
          callback();
        }, 3000);
      }

      salvarDadosBtn.addEventListener('click', () => {
        const nome = document.getElementById('nome').value.trim();
        const turma = document.getElementById('turma').value.trim();
        const ano = document.getElementById('ano').value.trim();

        if (!nome || !turma || !ano) {
          alert("Por favor, preencha todos os campos antes de começar!");
          return;
        }

        jogador = { nome, turma, ano, acertos: 0, erros: 0 };
        acertos = 0;
        erros = 0;
        perguntasRespondidas = 0;
        perguntasRestantes = [...perguntas];

        atualizarPontuacao();

        dadosJogadorDiv.style.display = 'none';
        salvarDadosBtn.style.display = 'none';

        roletaContainer.style.display = 'block';
        classificacaoBtn.style.display = 'none';

        selecionarPerguntaAleatoria();
      });

      function responderPergunta(respostaUsuario) {
        if (!perguntaAtual) return;

        if (respostaUsuario === perguntaAtual.resposta) {
          acertos++;
          respostaTexto.textContent = "Resposta correta!";
          respostaTexto.style.backgroundColor = "#4CAF50";
        } else {
          erros++;
          respostaTexto.textContent = "Resposta errada!";
          respostaTexto.style.backgroundColor = "#f44336";
        }

        perguntasRespondidas++;
        jogador.acertos = acertos;
        jogador.erros = erros;

        atualizarPontuacao();

        botoesResposta.style.display = 'none';

        // Girar roleta antes de próxima pergunta
        girarRoleta(() => {
          selecionarPerguntaAleatoria();
        });
      }

      document.getElementById('btn-verdadeiro').addEventListener('click', () => responderPergunta("Verdadeiro"));
      document.getElementById('btn-falso').addEventListener('click', () => responderPergunta("Falso"));

      classificacaoBtn.addEventListener('click', () => {
        // Mostrar tabela de classificação
        jogoContainer.style.display = 'none';
        classificacaoContainer.style.display = 'block';

        // Limpa tabela
        tabelaJogadores.innerHTML = "";

        // Buscar classificação do Google Sheets
        fetch(SHEET_ENDPOINT)
          .then(res => res.json())
          .then(data => {
            if (data.jogadores && data.jogadores.length > 0) {
              data.jogadores.sort((a, b) => b.acertos - a.acertos || a.erros - b.erros);
              data.jogadores.forEach(jogador => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                  <td>${jogador.nome}</td>
                  <td>${jogador.turma}</td>
                  <td>${jogador.ano}</td>
                  <td>${jogador.acertos}</td>
                  <td>${jogador.erros}</td>
                `;
                tabelaJogadores.appendChild(tr);
              });
            } else {
              tabelaJogadores.innerHTML = "<tr><td colspan='5'>Nenhum dado disponível</td></tr>";
            }
          })
          .catch(err => {
            tabelaJogadores.innerHTML = "<tr><td colspan='5'>Erro ao carregar dados</td></tr>";
            console.error("Erro ao buscar classificação:", err);
          });
      });

      btnVoltarJogo.addEventListener('click', () => {
        classificacaoContainer.style.display = 'none';
        jogoContainer.style.display = 'block';
      });
    });
  </script>
</body>
</html>
