# To upload PDF files in a React.js, Node.js, using MongoDB Atlas project with Multer and deploy on Render.com, follow these steps:

## Folder Structure

```
project-root
├── backend
│   ├── controllers
│   │   └── fileController.js
│   ├── models
│   │   └── fileModel.js
│   ├── routes
│   │   └── fileRoutes.js
│   ├── uploads
│   ├── .env
│   ├── server.js
│   ├── package.json
│   └── package-lock.json
├── frontend
│   ├── public
│   │   └── index.html
│   ├── src
│   │   ├── components
│   │   │   ├── FileUpload.js
│   │   │   └── FileView.js
│   │   ├── App.js
│   │   ├── index.js
│   ├── package.json
│   └── package-lock.json
└── README.md
```

## Backend Setup

### 1. Initialize a Node.js project:

>- mkdir backend
>- cd backend
>- npm init -y

### 1.2 mkdir backend
>- cd backend
>- npm init -y
>- npm install express mongoose multer dotenv cors

>- npm install multer@1.4.2


### Setup environment variables in .env:

>- MONGO_URI=mongodb+srv://teamproject440:qV7DQ8DFxpK0CBfO@cluster0.azogztt.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
PORT=5000



### Create server.js

```bash
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const cors = require('cors');
const fileRoutes = require('./routes/fileRoutes');

dotenv.config();

const app = express();

// CORS configuration
app.use(cors({
  origin: 'https://pdf-1-frontend.onrender.com', // Your frontend URL
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

app.use(express.json());
app.use('/uploads', express.static('uploads'));
app.use('/api/files', fileRoutes);

// Root route
app.get('/', (req, res) => {
  res.send('PDF Upload Server is running');
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});

mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### Create models/fileModel.js

```bash
const mongoose = require('mongoose');

const fileSchema = new mongoose.Schema({
  fileName: { type: String, required: true },
  filePath: { type: String, required: true }
});

module.exports = mongoose.model('File', fileSchema);

```


### Create controllers/fileController.js

```bash
const File = require('../model/fileModel');
const path = require('path');

exports.uploadFile = async (req, res) => {
  try {
    const newFile = new File({
      fileName: req.file.originalname,
      filePath: req.file.path
    });
    await newFile.save();
    res.status(201).json(newFile);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

exports.getFiles = async (req, res) => {
  try {
    const files = await File.find();
    res.status(200).json(files);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

```


### Create routes/fileRoutes.js

```bash
const express = require('express');
const multer = require('multer');
const path = require('path');
const { uploadFile, getFiles } = require('../Controllers/fileController');

const router = express.Router();

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  }
});

const upload = multer({ storage });

router.post('/upload', upload.single('file'), uploadFile);
router.get('/view', getFiles);

module.exports = router;
```


# Frontend Setup


# 1.1 Initialize a React project:

> - npx create-react-app frontend
> - cd frontend


# 1.2 Install dependencies:

>- npm install axios



###  1.3 Add the Babel plugin to your devDependencies:

>- npm install --save-dev @babel/plugin-proposal-private-property-in-object

###  1.4 Create or Update Babel Configuration:

- If you don't already have a Babel configuration file (babel.config.js or .babelrc), create one. Here, we'll use babel.config.js:

```bash
// babel.config.js
module.exports = {
  presets: ['react-app'],
  plugins: ['@babel/plugin-proposal-private-property-in-object']
};

```

### 1.5 Update package.json (Optional):

- If you want to ensure that your Babel configuration is picked up, you can add a Babel configuration section to your package.json:


```json
{
  "babel": {
    "presets": ["react-app"],
    "plugins": ["@babel/plugin-proposal-private-property-in-object"]
  }
}

```

### 1.6 Install update dependencies using below commands

>- npm install



## 2.1 Create src/components/FileUpload.js


```javascript
import React, { useState } from 'react';
import axios from 'axios';

const FileUpload = () => {
  const [file, setFile] = useState(null);
  const [uploadedFile, setUploadedFile] = useState(null);

  const onFileChange = (e) => {
    setFile(e.target.files[0]);
  };

  const onFileUpload = async () => {
    if (!file) {
      alert('Please select a file first.');
      return;
    }

    const formData = new FormData();
    formData.append('file', file);

    try {
      const res = await axios.post('https://pdf-sk0s.onrender.com/api/files/upload', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      });
      setUploadedFile(res.data);
      alert('File uploaded successfully.');
    } catch (error) {
      console.error('Upload error:', error);
      alert('Error uploading file: ' + (error.response?.data || error.message));
    }
  };

  const onViewResume = () => {
    if (uploadedFile) {
      window.open(`https://pdf-sk0s.onrender.com/uploads/${uploadedFile.filePath}`, '_blank');
    } else {
      alert('No file uploaded yet.');
    }
  };

  return (
    <div>
      <input type="file" onChange={onFileChange} />
      <button onClick={onFileUpload}>Upload Resume</button>
      <button onClick={onViewResume}>View Resume</button>
    </div>
  );
};

export default FileUpload;

```


## 2.2 Create src/components/FileView.js

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const FileView = () => {
  const [files, setFiles] = useState([]);

  useEffect(() => {
    const fetchFiles = async () => {
      try {
        const res = await axios.get('https://pdf-sk0s.onrender.com/api/files/view');
        setFiles(res.data);
      } catch (error) {
        console.error('Fetch error:', error);
        alert('Error fetching files: ' + (error.response?.data || error.message));
      }
    };
    fetchFiles();
  }, []);

  return (
    <ul>
      {files.map(file => (
        <li key={file._id}>
          <a href={`https://pdf-sk0s.onrender.com/uploads/${file.filePath}`} target="_blank" rel="noopener noreferrer">
            {file.fileName}
          </a>
        </li>
      ))}
    </ul>
  );
};

export default FileView;
```

## 2.3 Update src/App.js

```javascript
import React from 'react';
import FileUpload from './components/FileUpload';
import FileView from './components/FileView';

const App = () => {
  return (
    <div>
      <h1>Resume Upload and View</h1>
      <FileUpload />
      <FileView />
    </div>
  );
};

export default App;


```

## 2.4 Update src/App.js

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);


```

# Deployment on Render.com

## 3.1 Create a new web service for the backend:

-  Create a new web service for the backend:
- Go to Render.com.
- Create a new account or log in.
- Click on "New Web Service" and connect your GitHub repository.
- Select the **backend** directory optional for the root.
- Build **command backend/ $ npm install**
- start command **backend/ $ node server.js**
- Set the **environment variables (MONGO_URI, PORT)** in the Render dashboard or scroll below add env file copy paste from **.env file** add click button add  variable
- Deploy the backend.

>- **Note: If found any error check logs or build logs render settings modified code accordingly** 

## 3.2 Create a new static site for the frontend:

- Go to the Render dashboard.
- Click on "New Static Site" and connect your GitHub repository.
- Select the frontend directory optional  for the root.
- Build **command frontend/ $ npm install; npm run build**
- Publish Directory **frontend/ $ ./build** if your using dist use dist instead of build directory
- Kept rest setting as its default 
- Deploy the frontend.

>- **Note** If you found front end error after deploying functionality not working properly than press ctrl+shift+i inspect console paste console error in AI tool check for solution but check front deploy build logs 

# Reference Link 

- https://www.youtube.com/watch?v=cVEOhgPziO8


