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
