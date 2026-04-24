Código Front-End
Arquivo: index.html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Sistema ERP - Vendas</title>
  <style>
    body {
      font-family: Arial;
      background: #f4f4f4;
      padding: 20px;
    }

    h1 {
      text-align: center;
    }

    .container {
      max-width: 600px;
      margin: auto;
      background: white;
      padding: 20px;
      border-radius: 10px;
    }

    input, button {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
    }

    button {
      background: #007bff;
      color: white;
      border: none;
      cursor: pointer;
    }

    button:hover {
      background: #0056b3;
    }

    .venda {
      background: #eee;
      padding: 10px;
      margin-top: 10px;
      border-radius: 5px;
    }
  </style>
</head>
<body>

  <h1>Cadastro de Vendas</h1>

  <div class="container">
    <input type="text" id="cliente" placeholder="Nome do cliente">
    <input type="number" id="valor" placeholder="Valor da venda">
    <button onclick="criarVenda()">Cadastrar Venda</button>

    <h2>Lista de Vendas</h2>
    <div id="lista"></div>
  </div>

  <script>
    const API_URL = "http://localhost:3000"; // backend

    async function criarVenda() {
      const cliente = document.getElementById("cliente").value;
      const valor = document.getElementById("valor").value;

      if (!cliente || !valor) {
        alert("Preencha todos os campos!");
        return;
      }

      const resposta = await fetch(API_URL + "/vendas", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ cliente, valor })
      });

      if (resposta.ok) {
        alert("Venda criada com sucesso!");
        carregarVendas();
      } else {
        alert("Erro ao criar venda");
      }
    }

    async function carregarVendas() {
      const resposta = await fetch(API_URL + "/vendas");
      const vendas = await resposta.json();

      const lista = document.getElementById("lista");
      lista.innerHTML = "";

      vendas.forEach(v => {
        lista.innerHTML += `
          <div class="venda">
            <strong>${v.cliente}</strong><br>
            Valor: R$ ${v.valor}
          </div>
        `;
      });
    }

    carregarVendas();
  </script>

</body>
</html>

//


Back-end
Node.js + Express
criação projeto
mkdir erp-backend
cd erp-backend
npm init -y

instalação dependências
npm install express cors
Código do servidor
arquivo: server.js
const express = require('express');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

let vendas = [];
let ordens = [];

let idVenda = 1;
let idOrdem = 1;

//Criar venda
app.post('/vendas', (req, res) => {
  const { cliente, valor } = req.body;

  if (!cliente || !valor) {
    return res.status(400).json({ erro: 'Dados inválidos' });
  }

  const novaVenda = {
    id: idVenda++,
    cliente,
    valor,
    status: 'criada'
  };

  vendas.push(novaVenda);

  //REGRA DE NEGÓCIO: criar ordem automaticamente
  const ordem = {
    id: idOrdem++,
    venda_id: novaVenda.id,
    status: 'pendente'
  };

  ordens.push(ordem);

  res.status(201).json(novaVenda);
});

// Listar vendas
app.get('/vendas', (req, res) => {
  res.json(vendas);
});

// Listar ordens de serviço
app.get('/ordens', (req, res) => {
  res.json(ordens);
});

// iniciar servidor
app.listen(3000, () => {
  console.log('Servidor rodando em http://localhost:3000');
});


Repository (dados)
repositories/vendaRepository.js
let vendas = [];
let idVenda = 1;

function salvar(venda) {
  venda.id = idVenda++;
  vendas.push(venda);
  return venda;
}

function listar() {
  return vendas;
}

module.exports = { salvar, listar };
Service (regra de negócio)
Crie: services/vendaService.js
const vendaRepository = require('../repositories/vendaRepository');

let ordens = [];
let idOrdem = 1;

function criarVenda(dados) {
  if (!dados.cliente || !dados.valor) {
    throw new Error("Dados inválidos");
  }

  const venda = vendaRepository.salvar({
    cliente: dados.cliente,
    valor: dados.valor,
    status: "criada"
  });

  // REGRA DE NEGÓCIO
  const ordem = {
    id: idOrdem++,
    venda_id: venda.id,
    status: "pendente"
  };

  ordens.push(ordem);

  return venda;
}

function listarVendas() {
  return vendaRepository.listar();
}

function listarOrdens() {
  return ordens;
}

module.exports = { criarVenda, listarVendas, listarOrdens };
Controller (entrada da API)
Crie: controllers/vendaController.js
const vendaService = require('../services/vendaService');

function criar(req, res) {
  try {
    const venda = vendaService.criarVenda(req.body);
    res.status(201).json(venda);
  } catch (erro) {
    res.status(400).json({ erro: erro.message });
  }
}

function listar(req, res) {
  const vendas = vendaService.listarVendas();
  res.json(vendas);
}

function listarOrdens(req, res) {
  const ordens = vendaService.listarOrdens();
  res.json(ordens);
}

module.exports = { criar, listar, listarOrdens };
Routes
Crie: routes/vendaRoutes.js
const express = require('express');
const router = express.Router();
const controller = require('../controllers/vendaController');

router.post('/vendas', controller.criar);
router.get('/vendas', controller.listar);
router.get('/ordens', controller.listarOrdens);

module.exports = router;

server.js 
Atualize seu server.js:
const express = require('express');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

const vendaRoutes = require('./routes/vendaRoutes');

app.use('/', vendaRoutes);

app.listen(3000, () => {
  console.log("Servidor rodando em http://localhost:3000");
});
________________________________________
Banco de Dados: PostgreSQL
-- Usuários
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    senha VARCHAR(255) NOT NULL,
    perfil VARCHAR(50) NOT NULL,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Vendas
CREATE TABLE vendas (
    id SERIAL PRIMARY KEY,
    cliente VARCHAR(100) NOT NULL,
    valor NUMERIC(10,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'criada',
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Ordens de Serviço
CREATE TABLE ordens_servico (
    id SERIAL PRIMARY KEY,
    venda_id INT NOT NULL,
    descricao TEXT,
    status VARCHAR(50) DEFAULT 'pendente',
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (venda_id) REFERENCES vendas(id)
);

-- Faturas
CREATE TABLE faturas (
    id SERIAL PRIMARY KEY,
    venda_id INT NOT NULL,
    valor NUMERIC(10,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'pendente',
    data_emissao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (venda_id) REFERENCES vendas(id)
);

-- Chamados
CREATE TABLE chamados (
    id SERIAL PRIMARY KEY,
    titulo VARCHAR(100) NOT NULL,
    descricao TEXT,
    severidade VARCHAR(20),
    impacto VARCHAR(20),
    status VARCHAR(50) DEFAULT 'aberto',
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Logs (Auditoria)
CREATE TABLE logs (
    id SERIAL PRIMARY KEY,
    usuario_id INT,
    acao TEXT NOT NULL,
    data TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
Integração com Node.js
driver do PostgreSQL:
npm install pg
 Conexão com banco
database.js
const { Pool } = require('pg');

const pool = new Pool({
  user: 'postgres',
  host: 'localhost',
  database: 'erp',
  password: '1234',
  port: 5432,
});

module.exports = pool;
Repository usando banco
 vendaRepository.js
const db = require('../database');

async function salvar(venda) {
  const result = await db.query(
    'INSERT INTO vendas (cliente, valor) VALUES ($1, $2) RETURNING *',
    [venda.cliente, venda.valor]
  );
  return result.rows[0];
}

async function listar() {
  const result = await db.query('SELECT * FROM vendas');
  return result.rows;
}

module.exports = { salvar, listar };
________________________________________
regra de negócio no banco

await db.query('BEGIN');

const venda = await vendaRepository.salvar(dados);

await db.query(
  'INSERT INTO ordens_servico (venda_id) VALUES ($1)',
  [venda.id]
);

await db.query(
  'INSERT INTO faturas (venda_id, valor) VALUES ($1, $2)',
  [venda.id, venda.valor]
);

await db.query('COMMIT');

@agdelira
