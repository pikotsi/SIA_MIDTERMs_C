// server.js
const express = require('express');
const mysql = require('mysql2');
const app = express();
const PORT = 3001;

// Middleware
app.use(express.json());

// MySQL Connection
const db = mysql.createConnection({
  host: 'localhost',
  user: 'root',       
  password: 'user123',       
  database: 'store_db_c'
});

db.connect(err => {
  if (err) {
    console.error('Database connection failed:', err);
    return;
  }
  console.log('Connected to MySQL Database.');
});



app.get('/api/products', (req, res) => {
  const sql = 'SELECT * FROM products';
  db.query(sql, (err, results) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(results);
  });
});



app.get('/api/products/:id', (req, res) => {
  const { id } = req.params;
  const sql = 'SELECT * FROM products WHERE id = ?';
  db.query(sql, [id], (err, results) => {
    if (err) return res.status(500).json({ error: err.message });
    if (results.length === 0) return res.status(404).json({ message: 'Product not found' });
    res.json(results[0]);
  });
});



app.post('/api/products', (req, res) => {
  const { product_name, price } = req.body;
  if (!product_name || !price) {
    return res.status(400).json({ message: 'Missing required fields' });
  }

  const sql = 'INSERT INTO products (product_name, price) VALUES (?, ?)';
  db.query(sql, [product_name, price], (err, result) => {
    if (err) return res.status(500).json({ error: err.message });
    res.status(201).json({ message: 'Product added', id: result.insertId });
  });
});


app.put('/api/products/:id', (req, res) => {
  const { id } = req.params;
  const { product_name, price } = req.body;

  const sql = 'UPDATE products SET product_name = ?, price = ? WHERE id = ?';
  db.query(sql, [product_name, price, id], (err, result) => {
    if (err) return res.status(500).json({ error: err.message });
    if (result.affectedRows === 0) return res.status(404).json({ message: 'Product not found' });
    res.json({ message: 'Product updated successfully' });
  });
});



app.delete('/api/products/:id', (req, res) => {
  const { id } = req.params;
  const sql = 'DELETE FROM products WHERE id = ?';
  db.query(sql, [id], (err, result) => {
    if (err) return res.status(500).json({ error: err.message });
    if (result.affectedRows === 0) return res.status(404).json({ message: 'Product not found' });
    res.json({ message: 'Product deleted successfully' });
  });
});



app.get('/api/products/search', (req, res) => {
  const { name, minPrice } = req.query;
  let sql = 'SELECT * FROM products WHERE 1=1';
  const params = [];

  if (name) {
    sql += ' AND product_name LIKE ?';
    params.push(`%${name}%`);
  }
  if (minPrice) {
    sql += ' AND price >= ?';
    params.push(minPrice);
  }

  db.query(sql, params, (err, results) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(results);
  });
});



app.get('/api/products/details', (req, res) => {
  const sql = `
    SELECT p.id, p.product_name, p.price,
           c.category_name, s.supplier_name
    FROM products p
    JOIN categories c ON p.category_id = c.id
    JOIN suppliers s ON p.supplier_id = s.id
  `;

  db.query(sql, (err, results) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(results);
  });
});



app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
