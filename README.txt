const mysql = require('mysql');

// Create MySQL connection pool
const pool = mysql.createPool({
  connectionLimit: 10,
  host: 'localhost',
  user: 'your_username',
  password: 'your_password',
  database: 'your_database'
});

// Function to fetch data from database
function fetchData(query, params) {
  return new Promise((resolve, reject) => {
    pool.getConnection((err, connection) => {
      if (err) {
        reject(err);
        return;
      }

      connection.query(query, params, (error, results) => {
        connection.release();

        if (error) {
          reject(error);
          return;
        }

        resolve(results);
      });
    });
  });
}

// Example usage:
async function fetchUserByIdAndEmail(id, email) {
  try {
    const query = 'SELECT * FROM users WHERE id = ? AND email = ?';
    const params = [id, email];
    const users = await fetchData(query, params);
    console.log(users);
  } catch (error) {
    console.error('Error fetching user:', error);
  }
}

// Call the function
fetchUserByIdAndEmail(1, 'example@example.com');

===============================================================================================================================================================================
===============================================================================================================================================================================

const express = require('express');
const bodyParser = require('body-parser');
const mysql = require('mysql');
const csrf = require('csurf');

const app = express();
const port = 3000;

app.set('view engine', 'ejs');
app.use(express.static('public'));
app.use(bodyParser.urlencoded({ extended: true }));

// MySQL connection
const connection = mysql.createConnection({
    host: 'localhost',
    user: 'your_username',
    password: 'your_password',
    database: 'your_database'
});

connection.connect(err => {
    if (err) {
        console.error('Error connecting to MySQL:', err);
        return;
    }
    console.log('Connected to MySQL');
});

// CSRF protection
const csrfProtection = csrf({ cookie: true });

// Fetch data for update
app.get('/data/:id', (req, res) => {
    const userId = req.params.id;
    const query = 'SELECT * FROM users WHERE id = ?';
    connection.query(query, [userId], (error, results) => {
        if (error) {
            console.error('Error fetching data:', error);
            res.status(500).json({ error: 'Internal Server Error' });
            return;
        }
        if (results.length === 0) {
            res.status(404).json({ error: 'User not found' });
            return;
        }
        res.json(results[0]);
    });
});

// Update user data
app.post('/update', csrfProtection, (req, res) => {
    const { userId, name, email, phone } = req.body;
    const query = 'UPDATE users SET name = ?, email = ?, phone = ? WHERE id = ?';
    connection.query(query, [name, email, phone, userId], (error, results) => {
        if (error) {
            console.error('Error updating data:', error);
            res.status(500).json({ error: 'Internal Server Error' });
            return;
        }
        res.json({ message: 'Update successful' });
    });
});

app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});
