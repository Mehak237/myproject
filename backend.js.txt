const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'uploads')));

// Import Routes
const buyerRoutes = require('./routes/buyerRoutes');
const sellerRoutes = require('./routes/sellerRoutes');
const productRoutes = require('./routes/productRoutes');
const reviewRoutes = require('./routes/reviewRoutes');

// Routes
app.use('/api/buyer', buyerRoutes);
app.use('/api/seller', sellerRoutes);
app.use('/api/products', productRoutes);
app.use('/api/reviews', reviewRoutes);

// Home Route
app.get('/', (req, res) => {
    res.send('Welcome to Waste to Wonder Backend!');
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
const multer = require('multer');

// Set up storage engine for file uploads
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/'); // Save files to uploads folder
    },
    filename: (req, file, cb) => {
        cb(null, Date.now() + path.extname(file.originalname)); // Unique filename
    }
});

// File filter to only allow images
const fileFilter = (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
        cb(null, true);
    } else {
        cb(new Error('Not an image!'), false);
    }
};

const upload = multer({
    storage: storage,
    fileFilter: fileFilter
});

module.exports = upload;
const fs = require('fs');
const path = require('path');

// Helper function to read JSON data
const readJSONFile = (fileName) => {
    const filePath = path.join(__dirname, '../data', fileName);
    const rawData = fs.readFileSync(filePath, 'utf8');
    return JSON.parse(rawData);
};

// Helper function to write JSON data
const writeJSONFile = (fileName, data) => {
    const filePath = path.join(__dirname, '../data', fileName);
    fs.writeFileSync(filePath, JSON.stringify(data, null, 2));
};

module.exports = { readJSONFile, writeJSONFile };
const express = require('express');
const router = express.Router();
const { readJSONFile, writeJSONFile } = require('../utils/fileHandler');

router.post('/create', (req, res) => {
    const { name, email, contact, address } = req.body;
    const buyers = readJSONFile('buyers.json');
    const newBuyer = { name, email, contact, address };
    buyers.push(newBuyer);
    writeJSONFile('buyers.json', buyers);
    res.status(201).json({ message: 'Buyer profile created!', buyer: newBuyer });
});

module.exports = router;
const express = require('express');
const router = express.Router();
const { readJSONFile, writeJSONFile } = require('../utils/fileHandler');

router.post('/create', (req, res) => {
    const { storeName, storeDesc, email, contact } = req.body;
    const sellers = readJSONFile('sellers.json');
    const newSeller = { storeName, storeDesc, email, contact };
    sellers.push(newSeller);
    writeJSONFile('sellers.json', sellers);
    res.status(201).json({ message: 'Seller profile created!', seller: newSeller });
});

module.exports = router;
const express = require('express');
const router = express.Router();
const { readJSONFile, writeJSONFile } = require('../utils/fileHandler');
const upload = require('../middleware/upload');

router.post('/upload', upload.single('image'), (req, res) => {
    const { name, description, price } = req.body;
    const image = req.file ? req.file.filename : null;
    const products = readJSONFile('products.json');
    const newProduct = { name, description, price, image };
    products.push(newProduct);
    writeJSONFile('products.json', products);
    res.status(201).json({ message: 'Product uploaded!', product: newProduct });
});

router.get('/', (req, res) => {
    const products = readJSONFile('products.json');
    res.json(products);
});

module.exports = router;
const express = require('express');
const router = express.Router();
const { readJSONFile, writeJSONFile } = require('../utils/fileHandler');

router.post('/add', (req, res) => {
    const { productName, rating, review } = req.body;
    const reviews = readJSONFile('reviews.json');
    const newReview = { productName, rating, review };
    reviews.push(newReview);
    writeJSONFile('reviews.json', reviews);
    res.status(201).json({ message: 'Review added!', review: newReview });
});

module.exports = router;
