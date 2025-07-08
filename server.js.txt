const express = require('express');
const axios = require('axios');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

const CLIENT_ID = process.env.CLIENT_ID;
const CLIENT_SECRET = process.env.CLIENT_SECRET;

async function getToken() {
  const params = new URLSearchParams();
  params.append('client_id', CLIENT_ID);
  params.append('client_secret', CLIENT_SECRET);
  params.append('grant_type', 'client_credentials');

  const response = await axios.post('https://auth.mediroutes.com/connect/token', params);
  return response.data.access_token;
}

app.post('/submit', async (req, res) => {
  try {
    const token = await getToken();
    const rideData = req.body;

    const response = await axios.post('https://api.mediroutes.com/v1/rides', {
      name: rideData.firstName + ' ' + rideData.lastName,
      phone: rideData.phone,
      pickupLocation: rideData.pickup,
      dropoffLocation: rideData.dropoff,
      mobilityType: rideData.mobility,
      tripType: rideData.tripType,
      fundingSource: "Private Pay"
    }, {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    res.json({ success: true });
  } catch (error) {
    console.error(error?.response?.data || error.message);
    res.status(500).json({ error: 'Failed to submit ride request' });
  }
});

app.get('/', (req, res) => res.send('MediRoutes Proxy is running'));

app.listen(3000, () => console.log('Server running on port 3000'));
