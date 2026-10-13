# ========================================================
# Dockerfile demo phục vụ kiểm thử bài tập 4
# Khởi chạy một ứng dụng web nhẹ mô phỏng Spring Boot
# Cung cấp endpoint: /actuator/health trả về HTTP 200 {"status":"UP"}
# ========================================================
FROM python:3.11-alpine

WORKDIR /app

# Tạo script server mô phỏng Spring Boot Actuator
RUN cat <<'EOF' > server.py
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class HealthHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/actuator/health' or self.path == '/':
            self.send_response(200)
            self.send_header('Content-Type', 'application/json')
            self.end_headers()
            response = {
                "status": "UP",
                "components": {
                    "diskSpace": {"status": "UP"},
                    "ping": {"status": "UP"}
                },
                "service": "Spring Boot Demo Service"
            }
            self.wfile.write(json.dumps(response).encode('utf-8'))
        else:
            self.send_response(404)
            self.end_headers()

    def log_message(self, format, *args):
        # In log request ra console container
        print(f"[ACCESS LOG] {self.address_string()} - {format%args}")

if __name__ == '__main__':
    print("🚀 Spring Boot Mock Container listening on port 8080...")
    server = HTTPServer(('0.0.0.0', 8080), HealthHandler)
    server.serve_forever()
EOF

EXPOSE 8080

CMD ["python", "server.py"]

# ========================================================
# GHI CHÚ: Nếu bạn có dự án Spring Boot thực tế đã build ra file JAR:
# Thay thế nội dung trên bằng Dockerfile chuẩn dưới đây:
#
# FROM eclipse-temurin:17-jre-alpine
# WORKDIR /app
# COPY target/*.jar app.jar
# EXPOSE 8080
# ENTRYPOINT ["java", "-jar", "app.jar"]
# ========================================================
