# Error Handling Documentation

## Overview

SMon includes comprehensive error handling for both 404 (Not Found) and 500 (Internal Server Error) scenarios. The error handling system provides user-friendly HTML pages for browser requests and structured JSON responses for API calls.

## Error Types

### 404 - Not Found
Triggered when a requested route does not exist in the application.

**Browser Response**: Displays a styled HTML page with navigation options
**API Response**: Returns JSON with error details

### 500 - Internal Server Error
Triggered when an unexpected error occurs in the application.

**Browser Response**: Displays a styled HTML page with retry options and error tracking
**API Response**: Returns JSON with error details

## Features

### Smart Content Negotiation
- **HTML requests**: Served with styled error pages
- **API requests**: Served with JSON error responses
- **Automatic detection** based on `Accept` header

### Comprehensive Logging
All errors are logged with the following information:
- Error message and stack trace
- Request URL and method
- Client IP address
- User agent
- Timestamp

### Error Tracking
- **Error IDs**: Unique identifiers for tracking 500 errors
- **Timestamps**: When the error occurred
- **Request details**: URL, method, and client information

### User Experience
- **404 Page**: Clean design with navigation options
- **500 Page**: Includes retry functionality and auto-retry mechanism
- **Responsive design**: Works on all device sizes

## Error Pages

### Design Consistency
Error pages are designed to match the SMon application theme:
- **Framework**: Tailwind CSS for responsive design
- **Theme**: Dark gradient background (slate-900 to slate-800)
- **Colors**: Blue primary, orange/red accents, gray text
- **Icons**: Font Awesome icons
- **Components**: Card-based layout with rounded corners and shadows

### 404.html Features
- Orange-to-red gradient error icon
- Large "404" display
- "Go to Dashboard" and "Go Back" buttons
- Error details card with timestamp and requested URL
- Consistent with SMon's offline page design

### 500.html Features
- Red gradient error icon
- Large "500" display
- "Go to Dashboard" and "Try Again" buttons
- Retry section with additional retry button
- Error tracking with unique Error ID
- Auto-retry mechanism (optional)

## API Responses

### 404 Error Response
```json
{
  "error": {
    "message": "Route /nonexistent not found",
    "status": 404,
    "timestamp": "2024-01-01T12:00:00.000Z",
    "path": "/nonexistent"
  }
}
```

### 500 Error Response
```json
{
  "error": {
    "message": "Internal Server Error",
    "status": 500,
    "timestamp": "2024-01-01T12:00:00.000Z",
    "path": "/api/some-endpoint"
  }
}
```

## Testing

### Test Routes
Two test routes are available for development and testing:

- `GET /test-404` - Triggers a 404 error
- `GET /test-500` - Triggers a 500 error

**⚠️ Important**: Remove these routes in production!

### Testing Commands
```bash
# Test 404 error (HTML response)
curl -H "Accept: text/html" http://localhost:3000/test-404

# Test 404 error (JSON response)
curl -H "Accept: application/json" http://localhost:3000/test-404

# Test 500 error
curl http://localhost:3000/test-500

# Test with browser
open http://localhost:3000/test-404
open http://localhost:3000/test-500
```

## Implementation Details

### Middleware Order
Error handling middleware must be placed **after** all route definitions but **before** `app.listen()`:

```javascript
// 1. Static files middleware
app.use(express.static('public'));

// 2. Route definitions
app.get('/', (req, res) => { ... });
app.get('/api/data', (req, res) => { ... });

// 3. Error handling middleware (404 first, then 500)
app.use((req, res, next) => { /* 404 handler */ });
app.use((err, req, res, next) => { /* 500 handler */ });

// 4. Server start
app.listen(port, () => { ... });
```

### Error Object Structure
Custom errors should follow this pattern:

```javascript
const error = new Error('Custom error message');
error.status = 400; // Or any appropriate status code
next(error);
```

### Content Type Detection
The system automatically detects the appropriate response format:

```javascript
const acceptsHTML = req.accepts('html');
if (acceptsHTML) {
  // Send HTML page
  res.status(statusCode).sendFile(path.join(__dirname, 'public', '404.html'));
} else {
  // Send JSON response
  res.status(statusCode).json({ error: { ... } });
}
```

## Security Considerations

### Error Information Disclosure
- **Production**: Generic error messages (no stack traces)
- **Development**: Detailed error information for debugging
- **Logging**: Full error details logged server-side only

### Rate Limiting
Consider implementing rate limiting for error endpoints to prevent abuse:

```javascript
const rateLimit = require('express-rate-limit');
const errorLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/test-', errorLimiter); // Apply to test routes
```

## Customization

### Styling Error Pages
Error pages use embedded CSS for portability. To customize:

1. Edit the `<style>` sections in `404.html` and `500.html`
2. Update colors, fonts, and layout as needed
3. Test responsiveness on different devices

### Adding New Error Types
To add custom error pages:

1. Create new HTML file in `/public/` (e.g., `403.html`)
2. Update error handling middleware to serve the new page
3. Add appropriate status code handling

### Custom Error Messages
Modify error messages in the middleware:

```javascript
// Custom 404 message
const error = new Error('The page you are looking for does not exist');
error.status = 404;

// Custom 500 message
const error = new Error('Something went wrong. Please try again later');
error.status = 500;
```

## Monitoring and Alerting

### Log Analysis
Monitor error logs for patterns:

```bash
# View recent errors
tail -f logs/error.log | grep "Error occurred"

# Count errors by type
grep "status.*404" logs/error.log | wc -l
grep "status.*500" logs/error.log | wc -l
```

### Prometheus Metrics
Consider adding error metrics for monitoring:

```javascript
// Example error counter
const errorCounter = new client.Counter({
  name: 'smon_http_errors_total',
  help: 'Total number of HTTP errors',
  labelNames: ['status_code', 'path']
});

// In error handler
errorCounter.inc({ status_code: statusCode, path: req.path });
```

## Troubleshooting

### Common Issues

1. **Error pages not displaying**
   - Check if files exist in `/public/` directory
   - Verify file permissions
   - Ensure paths are correct in middleware

2. **JSON responses instead of HTML**
   - Check `Accept` header in request
   - Verify `req.accepts('html')` logic

3. **Errors not being logged**
   - Check console output permissions
   - Verify logging configuration
   - Ensure error handler is reached

4. **Test routes not working**
   - Confirm routes are defined before error handlers
   - Check for syntax errors in route definitions

### Debug Mode
Enable detailed error logging in development:

```javascript
if (process.env.NODE_ENV !== 'production') {
  // Log full error details
  console.error('Full error:', err);
}
```

## Production Deployment

### Cleanup
Before deploying to production:

1. **Remove test routes**:
   ```javascript
   // Remove these lines:
   app.get('/test-404', ...);
   app.get('/test-500', ...);
   ```

2. **Update error messages** for production:
   - Remove sensitive information
   - Use user-friendly language
   - Include support contact information

3. **Configure logging**:
   - Use structured logging (Winston, Bunyan)
   - Implement log rotation
   - Set up log aggregation (ELK stack, etc.)

### Performance Considerations
- Error pages are lightweight and cached by browsers
- JSON responses are minimal for API efficiency
- Logging is asynchronous to avoid blocking requests

## Support

For issues with error handling:

1. Check the logs for detailed error information
2. Test with the provided test routes
3. Verify middleware order in `app.js`
4. Ensure error page files exist and are accessible

## Changelog

### Version 1.0.0
- Initial implementation of error handling
- Added 404 and 500 error pages
- Implemented content negotiation
- Added comprehensive logging
- Created test routes for development