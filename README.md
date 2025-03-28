// Backend: server.js
const express = require('express');
const mongoose = require('mongoose');
const app = express();
const PORT = 3000;

// Connect to MongoDB
mongoose.connect('mongodb://localhost/socialmedia', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Middleware
app.use(express.json());

// Models
const Post = mongoose.model('Post', new mongoose.Schema({
  user: String,
  content: String,
  timestamp: { type: Date, default: Date.now },
}));

// Routes
app.post('/posts', async (req, res) => {
  const post = new Post(req.body);
  await post.save();
  res.send(post);
});

app.get('/posts', async (req, res) => {
  const posts = await Post.find();
  res.send(posts);
});

// Start Server
app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));


// Frontend: App.jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  const [posts, setPosts] = useState([]);
  const [content, setContent] = useState('');
  const [user, setUser] = useState('');

  useEffect(() => {
    axios.get('/posts').then(response => setPosts(response.data));
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    await axios.post('/posts', { user, content });
    setContent('');
    setUser('');
  };

  return (
    <div>
      <h1>Social Media App</h1>
      <form onSubmit={handleSubmit}>
        <input value={user} onChange={(e) => setUser(e.target.value)} placeholder="User" />
        <textarea value={content} onChange={(e) => setContent(e.target.value)} placeholder="What's on your mind?" />
        <button type="submit">Post</button>
      </form>
      <h2>Recent Posts</h2>
      {posts.map((post, index) => (
        <div key={index}>
          <h4>{post.user}</h4>
          <p>{post.content}</p>
          <small>{new Date(post.timestamp).toLocaleString()}</small>
        </div>
      ))}
    </div>
  );
}

export default App;
