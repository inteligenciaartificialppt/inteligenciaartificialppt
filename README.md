const express = require('express');
const bodyParser = require('body-parser');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const twilio = require('twilio');  // Para Twilio (si lo usarás)

const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Configuración de multer para la subida de imágenes
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/');
  },
  filename: function (req, file, cb) {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});
const upload = multer({ storage: storage });

// Simulación de base de datos
const messages = [];

// Ruta para recibir mensajes del formulario
app.post('/send-message', upload.single('image'), (req, res) => {
  const { name, employeeNumber, phone, message } = req.body;
  const image = req.file ? req.file.filename : null;

  // Guardar mensaje en la "base de datos"
  const newMessage = {
    name,
    employeeNumber,
    phone,
    message,
    image,
    response: null, // Aún no respondido
    timestamp: new Date()
  };

  messages.push(newMessage);
  console.log('Nuevo mensaje recibido:', newMessage);

  // Aquí puedes integrar Google Voice o Twilio para notificaciones
  // Twilio Ejemplo (debes conseguir tus credenciales de Twilio)
  // const accountSid = 'TU_ACCOUNT_SID';
  // const authToken = 'TU_AUTH_TOKEN';
  // const client = new twilio(accountSid, authToken);
  
  // client.messages.create({
  //    body: `Hola ${name}, tu mensaje ha sido recibido: ${message}`,
  //    to: phone,  // El teléfono del empleado
  //    from: '+1234567890' // Tu número de Twilio
  // }).then((message) => console.log(message.sid));

  res.json({ success: true, message: 'Mensaje recibido correctamente' });
});

// Ruta para responder a un mensaje
app.post('/respond-message', (req, res) => {
  const { employeeNumber, response } = req.body;
  
  // Buscar el mensaje por el número de empleado y responder
  const messageIndex = messages.findIndex(m => m.employeeNumber === employeeNumber);
  if (messageIndex !== -1) {
    messages[messageIndex].response = response;
    // Aquí podrías integrar una API para enviar la respuesta (Twilio o Google Voice)
    res.json({ success: true, message: 'Respuesta enviada correctamente' });
  } else {
    res.json({ success: false, message: 'Empleado no encontrado' });
  }
});

// Ruta para obtener los mensajes de un empleado
app.get('/messages/:employeeNumber', (req, res) => {
  const { employeeNumber } = req.params;
  const employeeMessages = messages.filter(m => m.employeeNumber === employeeNumber);
  res.json({ success: true, data: employeeMessages });
});

// Ruta para listar todos los mensajes
app.get('/messages', (req, res) => {
  res.json({ success: true, data: messages });
});

// Servidor
app.listen(3000, () => {
  console.log('Servidor corriendo en el puerto 3000');
});
