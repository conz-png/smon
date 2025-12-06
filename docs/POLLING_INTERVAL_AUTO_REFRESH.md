# Auto-Refresh Interval Configuration

## Overview

Semua halaman aplikasi SMon sekarang mengikuti **polling interval yang dikonfigurasi** di `settings.json` untuk auto-refresh data. Sebelumnya, setiap halaman memiliki hardcoded refresh interval (10 detik, 30 detik, dst) yang tidak sesuai dengan polling interval SNMP.

## Perubahan yang Dilakukan

### 1. Backend (app.js)

Menambahkan `pollingIntervalSeconds` ke semua route yang me-render views:

```javascript
const pollingIntervalSeconds = settings.pollingInterval / 1000;
const pingIntervalSeconds = settings.pingInterval ? settings.pingInterval / 1000 : 30;

res.render('page-name', { 
  // ... existing data ...
  pollingIntervalSeconds: pollingIntervalSeconds,
  pingIntervalSeconds: pingIntervalSeconds
});
```

**Routes yang diupdate:**
- `/` (Dashboard/Index)
- `/monitoring` 
- `/history`
- `/status`
- `/ping`
- `/about`
- `/domains`
- `/websites`
- `/settings`

### 2. Frontend (EJS Views)

Setiap view file sekarang:
- Mendeklarasikan `pollingIntervalMs` variable dari backend
- Menggunakan `pollingIntervalMs` dalam semua `setInterval()` calls
- Menghilangkan hardcoded intervals

**Contoh:**

```javascript
// Dulu (HARDCODED):
setInterval(updateHeaderStats, 10000); // Update every 10 seconds
setInterval(loadWebsiteStatus, 30000); // Update every 30 seconds

// Sekarang (DYNAMIC):
let pollingIntervalMs = <%= pollingIntervalSeconds * 1000 %>;
setInterval(updateHeaderStats, pollingIntervalMs);
setInterval(loadWebsiteStatus, pollingIntervalMs);
```

**Views yang diupdate:**
- `views/index.ejs`
- `views/monitoring.ejs`
- `views/history.ejs`
- `views/status.ejs`
- `views/ping.ejs`
- `views/about.ejs`
- `views/domains.ejs`
- `views/websites.ejs`

## Konfigurasi

### settings.json

```json
{
  "pollingInterval": 300000,    // 5 minutes (dalam milliseconds)
  "pingInterval": 120000,       // 2 minutes
  ...
}
```

### Mengubah Polling Interval

Edit `settings.json`:

```json
{
  "pollingInterval": 60000,     // Change to 1 minute
  "pollingInterval": 120000,    // Change to 2 minutes
  "pollingInterval": 300000,    // Change to 5 minutes (default)
}
```

Restart service:

```bash
pm2 restart smon
```

Halaman akan otomatis refresh dengan interval baru!

## Interval Reference

| Deskripsi | Milliseconds | Minutes |
|-----------|------------|---------|
| 30 detik | 30000 | 0.5 |
| 1 menit | 60000 | 1 |
| 2 menit | 120000 | 2 |
| 5 menit | 300000 | 5 (default) |
| 10 menit | 600000 | 10 |
| 15 menit | 900000 | 15 |
| 30 menit | 1800000 | 30 |

## Behavior

### Sebelum (Lama)

| Halaman | Update Interval |
|---------|----------------|
| Dashboard | 30 detik |
| Monitoring | 10 detik |
| History | 10 detik |
| Status | 30 detik |
| Ping | 10 detik |
| About | 10 detik |
| Domains | 30 detik |
| Websites | 30 detik |

**Masalah:**
- Setiap halaman refresh dengan interval berbeda
- Tidak sesuai dengan polling interval SNMP (biasanya 5 menit)
- Data berpotensi loading lebih sering dari SNMP polling

### Sesudah (Baru)

| Halaman | Update Interval |
|---------|----------------|
| Semua halaman | Mengikuti `settings.pollingInterval` |

**Keuntungan:**
✅ Konsisten di semua halaman
✅ Data selalu fresh setelah SNMP polling baru
✅ Tidak ada loading yang premature
✅ Dapat dikonfigurasi tanpa mengubah code
✅ Menghemat bandwidth/resource jika interval diperbesar

## Contoh Konfigurasi

### Skenario 1: Monitoring Tinggi (Real-time)

```json
{
  "pollingInterval": 60000,      // Poll setiap 1 menit
  "pingInterval": 30000          // Ping setiap 30 detik
}
```

Dashboard akan refresh setiap 1 menit dengan data terbaru.

### Skenario 2: Monitoring Normal (Default)

```json
{
  "pollingInterval": 300000,     // Poll setiap 5 menit
  "pingInterval": 120000         // Ping setiap 2 menit
}
```

Dashboard akan refresh setiap 5 menit.

### Skenario 3: Monitoring Ringan (Low Resource)

```json
{
  "pollingInterval": 900000,     // Poll setiap 15 menit
  "pingInterval": 300000         // Ping setiap 5 menit
}
```

Dashboard akan refresh setiap 15 menit, menghemat resource.

## Testing

Verify polling interval dengan membuka browser DevTools → Console:

```javascript
// Check current polling interval
console.log('Polling Interval:', pollingIntervalMs, 'ms');
console.log('Polling Interval:', pollingIntervalMs / 1000, 'seconds');
```

Atau gunakan test command:

```bash
# Check index page
curl -b cookies.txt http://localhost:3000/ | grep "pollingIntervalMs"

# Check specific page
curl -b cookies.txt http://localhost:3000/monitoring | grep "pollingIntervalMs"
```

## Catatan Teknis

### Update Interval Hierarchy

Beberapa komponen memiliki interval sendiri:

1. **Header System Stats** - Menggunakan `pollingIntervalMs`
2. **Device/Interface Data** - Menggunakan `pollingIntervalMs`
3. **Ping Monitoring** - Menggunakan `pingInterval` (bisa berbeda dari polling interval)
4. **Website Monitoring** - Menggunakan `pollingIntervalMs`
5. **Domain Expiry** - Menggunakan `pollingIntervalMs`

### Performance Impact

Mengurangi polling interval meningkatkan:
- ✅ Data freshness
- ❌ CPU usage
- ❌ Memory usage
- ❌ Network traffic
- ❌ Database writes

Menambah polling interval mengurangi resource usage tapi mengurangi real-time view.

### Default Recommendations

| Environment | Recommended Interval |
|-------------|---------------------|
| Development/Lab | 30-60 detik |
| Production (Few Devices) | 1-2 menit |
| Production (Many Devices) | 5-10 menit |
| Production (100+ Devices) | 10-15 menit |

## Troubleshooting

### Pages tidak auto-refresh

**Solusi:**
1. Clear browser cache: Ctrl+Shift+Delete
2. Hard refresh page: Ctrl+Shift+R
3. Check console untuk error: F12 → Console
4. Verify settings.json: `cat settings.json | head -1`

### Polling interval tidak berubah setelah edit settings.json

**Solusi:**
Restart service diperlukan:
```bash
pm2 restart smon
```

Atau reload page di browser untuk ambil setting terbaru.

### Resource usage terlalu tinggi

**Solusi:**
Kurangi polling interval:
```json
{
  "pollingInterval": 600000  // Naik dari 300000 ke 600000 (10 menit)
}
```

## Git Commits

Perubahan ini termasuk dalam commit:
```
Update auto-refresh intervals to follow configured polling interval
- All pages now respect settings.pollingInterval
- Removed hardcoded intervals (10s, 30s)
- Dynamic configuration per page
- Backend passes pollingIntervalSeconds to all views
```

## Next Enhancements

Fitur yang bisa dikembangkan:
- [ ] Per-page refresh interval override
- [ ] Adaptive polling (bertambah saat idle, berkurang saat aktif)
- [ ] Background service worker untuk sync
- [ ] Push notifications untuk critical events
- [ ] Database optimization untuk high-frequency polling

---

**Last Updated:** December 4, 2025
**Status:** Production Ready ✅