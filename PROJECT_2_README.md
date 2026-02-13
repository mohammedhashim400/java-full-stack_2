# Task Management - Notification Microservice

![Java](https://img.shields.io/badge/Java-17-orange?style=flat&logo=java)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-green?style=flat&logo=spring-boot)
![WebSocket](https://img.shields.io/badge/WebSocket-STOMP-blue?style=flat)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)

A standalone notification microservice extracted from the Task Management System, handling real-time notifications, email alerts, and notification preferences.

## 📖 Overview

This microservice handles all notification-related functionality for the Task Management System, including:
- Email notifications for task assignments and deadlines
- Real-time WebSocket notifications
- User notification preferences management
- Notification history and read status tracking
- Retry logic for failed deliveries

## ✨ Features

- ✅ **Multi-Channel Notifications**: Email and WebSocket (in-app)
- ✅ **Smart Scheduling**: Deadline reminders (24hr and 1hr before)
- ✅ **Retry Mechanism**: Exponential backoff for failed email delivery
- ✅ **Notification Preferences**: User-configurable notification settings
- ✅ **Real-time Updates**: WebSocket-based instant notifications
- ✅ **Notification History**: Track all notifications sent
- ✅ **Email Templates**: Customizable HTML email templates
- ✅ **Async Processing**: Non-blocking notification delivery

## 🛠️ Tech Stack

- **Java 17** with Spring Boot 3.x
- **Spring WebSocket** for real-time notifications
- **JavaMail API** for email delivery
- **Spring Scheduler** for deadline reminders
- **Redis** for notification queuing
- **MySQL** for notification history
- **Thymeleaf** for email templates

## 🚀 Quick Start

### Prerequisites

- JDK 17+
- Maven 3.6+
- Redis Server
- MySQL 8.0+
- SMTP server access (Gmail, SendGrid, etc.)

### Installation

```bash
# Clone repository
git clone https://github.com/YOUR_USERNAME/task-management-notification-service.git
cd task-management-notification-service

# Configure application properties
cp src/main/resources/application.properties.example src/main/resources/application.properties
# Edit application.properties with your settings

# Build project
mvn clean install

# Run service
mvn spring-boot:run
```

The service will start on `http://localhost:8081`

## ⚙️ Configuration

### application.properties

```properties
# Server Configuration
server.port=8081

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/notifications
spring.datasource.username=root
spring.datasource.password=your_password

# Redis Configuration
spring.redis.host=localhost
spring.redis.port=6379

# Email Configuration
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

# Notification Settings
notification.email.from=noreply@taskmanagement.com
notification.retry.max-attempts=3
notification.retry.delay-ms=5000

# WebSocket Configuration
websocket.endpoint=/ws
websocket.topic=/topic/notifications
```

## 📡 API Endpoints

### Notification Management

```
POST   /api/notifications/send              - Send notification
GET    /api/notifications                   - Get user notifications
GET    /api/notifications/{id}              - Get specific notification
PUT    /api/notifications/{id}/read         - Mark as read
DELETE /api/notifications/{id}              - Delete notification
GET    /api/notifications/unread/count      - Get unread count
```

### Preferences

```
GET    /api/notifications/preferences       - Get user preferences
PUT    /api/notifications/preferences       - Update preferences
```

### WebSocket

```
CONNECT /ws                                  - Establish WebSocket connection
SUBSCRIBE /topic/notifications/{userId}     - Subscribe to user notifications
```

## 📨 Notification Types

### 1. Task Assignment
Triggered when a task is assigned to a user.

```json
{
  "type": "TASK_ASSIGNED",
  "userId": 123,
  "taskId": 456,
  "title": "New Task Assignment",
  "message": "You have been assigned: Implement user authentication",
  "priority": "HIGH",
  "channels": ["EMAIL", "WEBSOCKET"]
}
```

### 2. Deadline Reminder
Sent 24 hours and 1 hour before task deadline.

```json
{
  "type": "DEADLINE_REMINDER",
  "userId": 123,
  "taskId": 456,
  "title": "Task Deadline Approaching",
  "message": "Task 'Deploy to production' is due in 1 hour",
  "priority": "URGENT",
  "channels": ["EMAIL", "WEBSOCKET"]
}
```

### 3. Status Change
Notifies when task status is updated.

```json
{
  "type": "STATUS_CHANGED",
  "userId": 123,
  "taskId": 456,
  "title": "Task Status Updated",
  "message": "Task 'Fix login bug' moved to In Review",
  "priority": "MEDIUM",
  "channels": ["WEBSOCKET"]
}
```

### 4. Comment Mention
Alerts user when mentioned in a comment.

```json
{
  "type": "COMMENT_MENTION",
  "userId": 123,
  "taskId": 456,
  "commentId": 789,
  "title": "You were mentioned",
  "message": "@john mentioned you in a comment",
  "priority": "MEDIUM",
  "channels": ["EMAIL", "WEBSOCKET"]
}
```

## 🔔 WebSocket Integration

### Client-Side Example (JavaScript)

```javascript
// Establish WebSocket connection
const socket = new SockJS('http://localhost:8081/ws');
const stompClient = Stomp.over(socket);

stompClient.connect({
  'Authorization': 'Bearer YOUR_JWT_TOKEN'
}, function(frame) {
  console.log('Connected: ' + frame);
  
  // Subscribe to user-specific notifications
  stompClient.subscribe('/topic/notifications/123', function(notification) {
    const message = JSON.parse(notification.body);
    displayNotification(message);
  });
});

function displayNotification(notification) {
  // Show notification in UI
  console.log('New notification:', notification);
  showToast(notification.title, notification.message);
}
```

## 📧 Email Templates

Email templates use Thymeleaf and are located in `src/main/resources/templates/email/`.

### Available Templates:

- `task-assigned.html` - Task assignment notification
- `deadline-reminder.html` - Deadline reminder
- `status-changed.html` - Status update notification
- `comment-mention.html` - Comment mention notification

### Customizing Templates:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Task Assigned</title>
</head>
<body>
    <h1>New Task Assignment</h1>
    <p>Hello <span th:text="${userName}">User</span>,</p>
    <p>You have been assigned a new task:</p>
    <h2 th:text="${taskTitle}">Task Title</h2>
    <p th:text="${taskDescription}">Task description...</p>
    <p><strong>Priority:</strong> <span th:text="${priority}">High</span></p>
    <p><strong>Deadline:</strong> <span th:text="${deadline}">2024-12-31</span></p>
    <a th:href="${taskUrl}">View Task</a>
</body>
</html>
```

## 🔄 Retry Logic

Failed email notifications are automatically retried with exponential backoff:

```
Attempt 1: Immediate
Attempt 2: 5 seconds delay
Attempt 3: 25 seconds delay (5^2)
Attempt 4: 125 seconds delay (5^3)
```

After max attempts, the notification is marked as failed and logged.

## ⚡ Performance Features

- **Async Processing**: Notifications sent asynchronously
- **Batch Processing**: Multiple notifications queued and sent in batches
- **Redis Queue**: Prevents notification loss during system restarts
- **Connection Pooling**: Efficient SMTP connection management
- **Rate Limiting**: Prevents email spam

## 🧪 Testing

```bash
# Run all tests
mvn test

# Run specific test
mvn test -Dtest=NotificationServiceTest

# Integration tests
mvn verify
```

### Test Coverage:
- Unit Tests: Service layer (90% coverage)
- Integration Tests: API endpoints
- WebSocket Tests: Real-time notification delivery
- Email Tests: Mock SMTP server

## 📊 Monitoring

### Health Check Endpoint

```bash
GET /actuator/health
```

Response:
```json
{
  "status": "UP",
  "components": {
    "email": {
      "status": "UP",
      "details": {
        "smtp_host": "smtp.gmail.com",
        "connection": "healthy"
      }
    },
    "redis": {
      "status": "UP"
    },
    "websocket": {
      "status": "UP",
      "activeConnections": 42
    }
  }
}
```

### Metrics

```bash
GET /actuator/metrics/notifications.sent
GET /actuator/metrics/notifications.failed
GET /actuator/metrics/websocket.connections
```

## 📁 Project Structure

```
notification-service/
├── src/
│   ├── main/
│   │   ├── java/com/amdox/notification/
│   │   │   ├── controller/
│   │   │   │   ├── NotificationController.java
│   │   │   │   └── WebSocketController.java
│   │   │   ├── service/
│   │   │   │   ├── NotificationService.java
│   │   │   │   ├── EmailService.java
│   │   │   │   └── WebSocketService.java
│   │   │   ├── repository/
│   │   │   │   └── NotificationRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Notification.java
│   │   │   │   └── NotificationPreference.java
│   │   │   ├── dto/
│   │   │   │   └── NotificationRequest.java
│   │   │   ├── config/
│   │   │   │   ├── WebSocketConfig.java
│   │   │   │   └── EmailConfig.java
│   │   │   └── scheduler/
│   │   │       └── DeadlineReminderScheduler.java
│   │   └── resources/
│   │       ├── application.properties
│   │       └── templates/email/
│   │           ├── task-assigned.html
│   │           └── deadline-reminder.html
│   └── test/
├── pom.xml
└── README.md
```

## 🔧 Troubleshooting

### Email Not Sending

1. Check SMTP credentials in `application.properties`
2. Enable "Less secure app access" for Gmail (or use App Password)
3. Check firewall/network restrictions on SMTP port (587/465)
4. Review logs: `tail -f logs/notification-service.log`

### WebSocket Connection Issues

1. Verify Redis is running: `redis-cli ping`
2. Check CORS configuration if connecting from different domain
3. Ensure JWT token is valid and included in connection headers
4. Check browser console for connection errors

## 🚀 Deployment

### Docker

```dockerfile
FROM openjdk:17-alpine
COPY target/notification-service-1.0.0.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java","-jar","/app.jar"]
```

```bash
# Build image
docker build -t notification-service .

# Run container
docker run -p 8081:8081 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://host:3306/notifications \
  -e SPRING_MAIL_USERNAME=your-email@gmail.com \
  notification-service
```

## 📈 Future Enhancements

- [ ] Push notifications for mobile apps
- [ ] SMS notifications via Twilio
- [ ] Slack integration
- [ ] Notification analytics dashboard
- [ ] A/B testing for notification content
- [ ] Smart notification batching (digest mode)
- [ ] Multi-language support

## 📝 License

MIT License - see [LICENSE](LICENSE) file

## 👨‍💻 Author

**[Your Name]**
- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [Your Profile](https://linkedin.com/in/your-profile)

## 🙏 Acknowledgments

Part of the Task Management System project developed during internship at AMDOX.

---

⭐ Star this repo if you find it helpful!
