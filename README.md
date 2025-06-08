const express = require('express');
const http = require('http');
const cors = require('cors');
const { Server } = require('socket.io');
const path = require('path');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' }
});

app.use(cors());
app.use(express.json());
app.use(express.static('public'));

let storedContent = {
  thoughts: [],
  pickup: [],
  quotes: [],
  stories: [],
  ideas: [],
  news: [],
  trending: []
};

app.get('/fetch', (req, res) => {
  res.json(storedContent);
});

app.post('/save', (req, res) => {
  const data = req.body;
  Object.entries(data).forEach(([section, items]) => {
    if (storedContent[section]) {
      storedContent[section].push(...items);
      io.emit('newContent', { sectionId: section, item: items[0] });
    }
  });
  res.json({ success: true });
});

io.on('connection', (socket) => {
  console.log('🔗 A user connected');
  socket.on('newContent', ({ sectionId, item }) => {
    if (storedContent[sectionId]) {
      storedContent[sectionId].push(item);
      io.emit('newContent', { sectionId, item });
    }
  });
  socket.on('disconnect', () => {
    console.log('❌ User disconnected');
  });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => console.log(`🚀 Server running at http://localhost:${PORT}`));
