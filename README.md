mbm-coupon-api/
│
├── index.js
├── package.json
├── firebaseServiceAccountKey.json  ← (Firebase Admin SDK key)
└── README.md
const express = require('express');
const admin = require('firebase-admin');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
app.use(cors());
app.use(bodyParser.json());

const serviceAccount = require('./firebaseServiceAccountKey.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: 'https://<YOUR-FIREBASE-PROJECT>.firebaseio.com'
});

const db = admin.firestore();

app.post('/verify-coupon', async (req, res) => {
  const { couponCode, userId } = req.body;

  try {
    const doc = await db.collection('coupons').doc(couponCode).get();

    if (!doc.exists) {
      return res.status(404).json({ valid: false, message: 'Invalid coupon' });
    }

    const coupon = doc.data();

    if (coupon.usedBy) {
      return res.status(409).json({ valid: false, message: 'Already used' });
    }

    await db.collection('coupons').doc(couponCode).update({
      usedBy: userId,
      usedAt: new Date()
    });

    res.json({ valid: true, discount: coupon.discount });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(3000, () => {
  console.log('MBM Coupon API running on port 3000');
});
{
  "name": "mbm-coupon-api",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "firebase-admin": "^11.10.1",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  }
}
