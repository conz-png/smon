# SMon - SNMP Monitoring Dashboard

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-4.x-lightgrey.svg)](https://expressjs.com/)

**SMon** adalah sistem monitoring jaringan canggih yang menggunakan protokol SNMP (Simple Network Management Protocol) untuk memantau performa bandwidth dan status perangkat jaringan secara real-time. Aplikasi ini dirancang untuk memberikan visibilitas komprehensif terhadap infrastruktur jaringan dengan antarmuka yang user-friendly dan responsif.

## ✨ Fitur Utama

- 🔍 **Real-time Bandwidth Monitoring** - Pantau traffic RX/TX secara real-time
- 🖥️ **Ping Monitoring** - Monitor konektivitas jaringan dengan interval yang dapat dikonfigurasi
- 🖥️ **Multi-Device Support** - Monitor multiple network devices simultaneously
- 📱 **Responsive Design** - Akses dari desktop dan mobile device
- 📊 **Advanced Analytics** - Visualisasi data dengan Chart.js
- ⚙️ **Configurable Settings** - Interval polling yang dapat dikonfigurasi
- 🔐 **Authentication System** - Sistem login dengan session management
- 📈 **InfluxDB Integration** - Time-series database untuk penyimpanan data
- 🎨 **Modern UI** - Interface dengan dark theme yang elegan

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- InfluxDB 2.x
- SNMP-enabled network devices

### Installation

1. **Clone repository**
   ```bash
   git clone <repository-url>
   cd graphts
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure InfluxDB**
   - Setup InfluxDB dengan bucket `graphts`
   - Update token dan konfigurasi di `app.js`

4. **Configure Devices**
   - Edit `config.json` untuk menambahkan device SNMP
   - Pastikan community string dan IP address benar

5. **Start Application**
   ```bash
   npm start
   # atau menggunakan PM2
   pm2 start app.js --name smon
   ```

6. **Access Application**
   - Open browser: `http://localhost:3000`
   - Login dengan credentials default:
     - Username: `admin`
     - Password: `admin123`

## 📸 Screenshots

### Login Page
![Login Page](screenshots/SMon%20-%20Login.png)

Halaman login dengan autentikasi sederhana untuk mengakses dashboard monitoring.

### Dashboard
![Dashboard](screenshots/Smon%20-%20Dashboard.png)

Dashboard utama menampilkan overview sistem dan informasi real-time.

### Bandwidth Monitor
![Bandwidth Monitor](screenshots/Smon%20-%20Bandwidth%20Monitor.png)

Monitoring bandwidth real-time dengan grafik RX/TX untuk setiap interface.

### Ping Monitor
![Ping Monitor](screenshots/SMon%20-%20Ping%20Monitor.png)

Monitoring konektivitas jaringan dengan status real-time dan konfigurasi interval yang dapat disesuaikan.

### Devices Management
![Devices Management](screenshots/SMon%20-%20Devices.png)

Manajemen perangkat jaringan dengan konfigurasi SNMP interface.

### Settings
![Settings](screenshots/SMon%20-%20Settings.png)

Konfigurasi aplikasi termasuk interval polling dan pengaturan sistem.

### About Page
![About Page](screenshots/SMon%20-%20About.png)

Informasi tentang aplikasi dan dedikasi khusus.

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Browser   │────│   Express.js    │────│   InfluxDB      │
│   (Dashboard)   │    │   (Backend)     │    │   (Time-series) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │   SNMP Devices  │
                       │   (Network)     │
                       └─────────────────┘
```

### Technology Stack

- **Backend**: Node.js, Express.js
- **Database**: InfluxDB 2.x (Time-series)
- **Frontend**: EJS Templates, Tailwind CSS, Chart.js
- **Network**: Net-SNMP library
- **Authentication**: Cookie-based sessions
- **Process Management**: PM2

## 📁 Project Structure

```
graphts/
├── app.js                 # Main application file
├── config.json           # SNMP devices configuration
├── settings.json         # Application settings
├── ping-targets.json     # Ping monitoring targets
├── package.json          # Dependencies and scripts
├── views/                # EJS templates
│   ├── index.ejs        # Dashboard page
│   ├── monitoring.ejs   # Bandwidth monitor page
│   ├── ping.ejs         # Ping monitoring page
│   ├── devices.ejs      # Device management page
│   ├── settings.ejs     # Settings page
│   ├── login.ejs        # Login page
│   └── about.ejs        # About page
├── public/              # Static assets (CSS, JS, images)
├── docs/                # Documentation
│   ├── README.md       # This file
│   └── screenshots/    # Application screenshots
└── scripts/             # Utility scripts
```

## ⚙️ Configuration

### SNMP Devices Configuration (`config.json`)

```json
{
  "snmpDevices": [
    {
      "id": "core-router",
      "name": "Core Router",
      "host": "192.168.1.1",
      "community": "public",
      "enabled": true,
      "selectedInterfaces": [
        {
          "index": 1,
          "name": "ether1"
        }
      ]
    }
  ]
}
```

### Application Settings (`settings.json`)

```json
{
  "pollingInterval": 60000,
  "pingInterval": 30000,
  "dataRetention": 365
}
```

**Settings Configuration:**
- `pollingInterval`: SNMP polling interval in milliseconds (default: 60000 = 1 minute)
- `pingInterval`: Ping monitoring interval in milliseconds (default: 30000 = 30 seconds)
- `dataRetention`: Data retention period in days (default: 365, max: 730)

### Ping Targets Configuration (`ping-targets.json`)

```json
[
  {
    "id": 1,
    "name": "Google DNS",
    "host": "8.8.8.8",
    "group": "DNS",
    "enabled": true
  },
  {
    "id": 2,
    "name": "Cloudflare DNS",
    "host": "1.1.1.1",
    "group": "DNS",
    "enabled": true
  }
]
```

**Ping Target Properties:**
- `id`: Unique identifier for the target
- `name`: Display name for the target
- `host`: IP address or hostname to ping
- `group`: Grouping category (e.g., "DNS", "Network", "Servers")
- `enabled`: Whether the target is actively monitored

## 🔧 API Endpoints

### Authentication
- `GET /login` - Login page
- `POST /login` - Process login
- `GET /logout` - Logout

### Dashboard
- `GET /` - Main dashboard
- `GET /api/system-info` - System information

### Monitoring
- `GET /monitoring` - Bandwidth monitoring page
- `GET /api/data` - Bandwidth data API

### Ping Monitoring
- `GET /ping` - Ping monitoring page
- `GET /api/ping-targets` - Ping targets list API
- `POST /api/ping-targets` - Add ping target
- `DELETE /api/ping-targets/:id` - Delete ping target
- `PATCH /api/ping-targets/:id/toggle` - Enable/disable ping target
- `POST /api/ping-test` - Test ping connectivity

### Device Management
- `GET /devices` - Device management page
- `GET /api/devices` - Device list API

### Settings
- `GET /settings` - Settings page
- `GET /api/settings` - Settings API
- `POST /api/settings` - Update settings

### Information
- `GET /about` - About page

## 🔍 Monitoring Details

### SNMP OIDs Used
- **ifInOctets** (1.3.6.1.2.1.2.2.1.10) - Inbound traffic
- **ifOutOctets** (1.3.6.1.2.1.2.2.1.16) - Outbound traffic
- **ifDescr** (1.3.6.1.2.1.2.2.1.2) - Interface description
- **ifOperStatus** (1.3.6.1.2.1.2.2.1.8) - Interface status

### Data Processing
- Bandwidth calculation: `(octets * 8) / (1000000 * time_interval)`
- Derivative calculation for rate computation
- Zero value filtering for clean charts
- Time-series aggregation with InfluxDB

## 🚀 Deployment

### Using PM2 (Recommended)

```bash
# Install PM2 globally
npm install -g pm2

# Start application
pm2 start app.js --name smon

# Save PM2 configuration
pm2 save

# Setup auto-start on boot
pm2 startup
```

### Docker Deployment

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
EXPOSE 3000

CMD ["npm", "start"]
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Dedication

Aplikasi ini didedikasikan untuk **Almarhum Ananda M. Habibie El-Rizq**, yang selalu memberikan inspirasi dan motivasi dalam setiap langkah pengembangan teknologi dan inovasi.

*"Dalam kenangan yang abadi, inovasi terus berkembang"*

## 📞 Support

For support, email support@smon.local or join our Discord community.

## 🔄 Changelog

### v2.0 (November 2025)
- ✨ Added authentication system
- 🎨 Modern dark theme UI
- 📱 Responsive mobile design
- 🔐 Cookie-based session management
- 📊 Enhanced chart visualizations
- ⚙️ Configurable polling intervals
- 🖥️ Multi-device SNMP monitoring

### v1.0 (Initial Release)
- Basic SNMP monitoring functionality
- Simple dashboard interface
- InfluxDB integration

---

**Built with ❤️ using modern web technologies**