# sistema_reservas
Sistema de Reservas online

Estrutura de arquivos do projeto 

project-root/
│
├── app.js
├── package.json
├── config/
│   └── database.js
├── controllers/
│   ├── spaceController.js
│   └── reservationController.js
├── models/
│   ├── index.js
│   ├── space.js
│   └── reservation.js
├── routes/
│   ├── spaceRoutes.js
│   └── reservationRoutes.js
└── ...

App em Javascript

const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const db = require('./models');
const spaceRoutes = require('./routes/spaceRoutes');
const reservationRoutes = require('./routes/reservationRoutes');

const app = express();

app.use(cors());
app.use(bodyParser.json());
app.use('/spaces', spaceRoutes);
app.use('/reservations', reservationRoutes);

db.sequelize.sync().then(() => {
  app.listen(3000, () => {
    console.log('Server is running on port 3000');
  });
});

Configuração do Banco de dados 

module.exports = {
  development: {
    username: 'root',
    password: 'password',
    database: 'reservation_system',
    host: '127.0.0.1',
    dialect: 'mysql',
  },
  ...
};

Modelos (Models/Spaces.js)

module.exports = (sequelize, DataTypes) => {
  const Space = sequelize.define('Space', {
    name: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    capacity: {
      type: DataTypes.INTEGER,
      allowNull: false,
    },
  });

  Space.associate = (models) => {
    Space.hasMany(models.Reservation);
  };

  return Space;
};

Modelo de Reservas

module.exports = (sequelize, DataTypes) => {
  const Reservation = sequelize.define('Reservation', {
    date: {
      type: DataTypes.DATEONLY,
      allowNull: false,
    },
    startTime: {
      type: DataTypes.TIME,
      allowNull: false,
    },
    endTime: {
      type: DataTypes.TIME,
      allowNull: false,
    },
  });

  Reservation.associate = (models) => {
    Reservation.belongsTo(models.Space);
  };

  return Reservation;
}; 

Controladores de Espaço

const db = require('../models');

const getAllSpaces = async (req, res) => {
  const spaces = await db.Space.findAll();
  res.status(200).json(spaces);
};

const createSpace = async (req, res) => {
  const { name, capacity } = req.body;
  const newSpace = await db.Space.create({ name, capacity });
  res.status(201).json(newSpace);
};

module.exports = {
  getAllSpaces,
  createSpace,
};

Controladores de Reserva

const db = require('../models');

const getAllReservations = async (req, res) => {
  const reservations = await db.Reservation.findAll();
  res.status(200).json(reservations);
};

const createReservation = async (req, res) => {
  const { date, startTime, endTime, SpaceId } = req.body;

  // Verifique se há conflitos
  const conflicts = await db.Reservation.findOne({
    where: {
      SpaceId,
      date,
      [db.Sequelize.Op.or]: [
        {
          startTime: {
            [db.Sequelize.Op.between]: [startTime, endTime],
          },
        },
        {
          endTime: {
            [db.Sequelize.Op.between]: [startTime, endTime],
          },
        },
      ],
    },
  });

  if (conflicts) {
    return res.status(400).json({ message: 'Conflito de horário' });
  }

  const newReservation = await db.Reservation.create({ date, startTime, endTime, SpaceId });
  res.status(201).json(newReservation);
};

// Adicione outras funções conforme necessário (update, delete)

module.exports = {
  getAllReservations,
  createReservation,
};

Rotas Espaços

const express = require('express');
const router = express.Router();
const spaceController = require('../controllers/spaceController');

router.get('/', spaceController.getAllSpaces);
router.post('/', spaceController.createSpace);
module.exports = router;

Rotas Reservas

const express = require('express');
const router = express.Router();
const reservationController = require('../controllers/reservationController');

router.get('/', reservationController.getAllReservations);
router.post('/', reservationController.createReservation);

module.exports = router;

Frontend Javascript

Estrutura do projeto para o frontend desenvolvido em Javascript.
project-root/
│
├── index.html
├── styles.css
└── scripts.js

Html
Desenvolvimento em Html do front-end

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistema de Reservas</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1>Sistema de Reservas</h1>
  <div id="spaces">
    <h2>Espaços Disponíveis</h2>
    <ul id="space-list"></ul>
  </div>
  <div id="reservations">
    <h2>Minhas Reservas</h2>
    <ul id="reservation-list"></ul>
  </div>
  <script src="scripts.js"></script>
</body>
</html>

Javascript

Desenvolvimento do front end em script.js

const baseURL = 'http://localhost:3000';

// Função para buscar e exibir espaços
const fetchSpaces = async () => {
  const response = await fetch(`${baseURL}/spaces`);
  const spaces = await response.json();
  const spaceList = document.getElementById('space-list');

  spaces.forEach(space => {
    const li = document.createElement('li');
    li.textContent = `${space.name} - Capacidade: ${space.capacity}`;
    spaceList.appendChild(li);
  });
};

// Função para buscar e exibir reservas
const fetchReservations = async () => {
  const response = await fetch(`${baseURL}/reservations`);
  const reservations = await response.json();
  const reservationList = document.getElementById('reservation-list');

  reservations.forEach(reservation => {
    const li = document.createElement('li');
    li.textContent = `Espaço ID: ${reservation.SpaceId}, Data: ${reservation.date}, Início: ${reservation.startTime}, Fim: ${reservation.endTime}`;
    reservationList.appendChild(li);
  });
};

// Inicialize as funções ao carregar a página
document.addEventListener('DOMContentLoaded', () => {
  fetchSpaces();
  fetchReservations();
});

CSS

Desenvolvimento em CSS

body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
}

h1 {
  margin: 20px 0;
}

#spaces, #reservations {
  width: 80%;
  margin: 20px 0;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  background-color: #f9f9f9;
  margin: 10px 0;
  padding: 10px;
  border-radius: 5px;
}


Banco de Dados SQL

Tabelas utilizando  banco de dados SQL 

CREATE TABLE Spaces (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  capacity INT NOT NULL
);

CREATE TABLE Reservations (
  id INT AUTO_INCREMENT PRIMARY KEY,
  date DATE NOT NULL,
  startTime TIME NOT NULL,
  endTime TIME NOT NULL,
  SpaceId INT,
  FOREIGN KEY (SpaceId) REFERENCES Spaces(id)
);
