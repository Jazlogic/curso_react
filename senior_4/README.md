# 🚀 Módulo 10: DevOps y Deployment

## 📚 Descripción del Módulo

En este módulo aprenderás a implementar DevOps completo para aplicaciones React, incluyendo CI/CD, Docker, cloud deployment, monitoreo, y estrategias para llevar aplicaciones a producción de manera segura y escalable.

## 🎯 Objetivos de Aprendizaje

Al finalizar este módulo serás capaz de:
- ✅ Configurar pipelines de CI/CD completos
- ✅ Crear y gestionar contenedores Docker
- ✅ Implementar deployment en la nube (AWS, Azure, GCP)
- ✅ Configurar monitoreo y alertas de producción
- ✅ Implementar estrategias de deployment (Blue-Green, Canary)
- ✅ Gestionar variables de entorno y secretos
- ✅ Configurar CDN y optimización de assets
- ✅ Implementar backup y disaster recovery
- ✅ Gestionar múltiples entornos (dev, staging, prod)
- ✅ Crear estrategias de rollback y versionado

## 📖 Contenido Teórico

### 1. CI/CD con GitHub Actions

#### Pipeline completo de CI/CD:
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Testing y Quality Assurance
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linting
      run: npm run lint
    
    - name: Run type checking
      run: npm run type-check
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella
    
    - name: Run E2E tests
      run: npm run test:e2e
      env:
        CI: true

  # Security scanning
  security:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Run security audit
      run: npm audit --audit-level=moderate
    
    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high

  # Build y Docker
  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build application
      run: npm run build
      env:
        REACT_APP_API_URL: ${{ secrets.REACT_APP_API_URL }}
        REACT_APP_ENVIRONMENT: production
    
    - name: Build Docker image
      run: docker build -t ${{ env.IMAGE_NAME }}:${{ github.sha }} .
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Push Docker image
      run: |
        docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
        docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
    
    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: build-files
        path: dist/

  # Deployment a staging
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # Aquí irían los comandos de deployment a staging
    
    - name: Run smoke tests
      run: |
        echo "Running smoke tests on staging..."
        # Tests básicos para verificar que la app funciona

  # Deployment a producción
  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production environment..."
        # Deployment a producción con estrategia Blue-Green
    
    - name: Run health checks
      run: |
        echo "Running health checks..."
        # Verificar que la app esté funcionando correctamente
    
    - name: Notify deployment success
      run: |
        echo "Production deployment successful!"
        # Notificar al equipo sobre el deployment exitoso

  # Performance testing
  performance:
    needs: deploy-production
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Run Lighthouse CI
      uses: treosh/lighthouse-ci-action@v10
      with:
        urls: |
          https://myapp.com
          https://myapp.com/dashboard
        uploadArtifacts: true
        temporaryPublicStorage: true
        configPath: './lighthouserc.json'
```

#### Configuración de Lighthouse CI:
```json
// lighthouserc.json
{
  "ci": {
    "collect": {
      "url": [
        "https://myapp.com",
        "https://myapp.com/dashboard",
        "https://myapp.com/users"
      ],
      "numberOfRuns": 3,
      "settings": {
        "preset": "desktop",
        "chromeFlags": "--no-sandbox --disable-dev-shm-usage"
      }
    },
    "assert": {
      "assertions": {
        "categories:performance": ["warn", {"minScore": 0.8}],
        "categories:accessibility": ["error", {"minScore": 0.9}],
        "categories:best-practices": ["warn", {"minScore": 0.8}],
        "categories:seo": ["warn", {"minScore": 0.8}],
        "first-contentful-paint": ["warn", {"maxNumericValue": 2000}],
        "largest-contentful-paint": ["warn", {"maxNumericValue": 2500}],
        "cumulative-layout-shift": ["warn", {"maxNumericValue": 0.1}]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

### 2. Docker y Contenedores

#### Dockerfile optimizado para React:
```dockerfile
# Dockerfile
# Multi-stage build para optimizar el tamaño final
FROM node:18-alpine AS builder

# Instalar dependencias del sistema
RUN apk add --no-cache python3 make g++

# Establecer directorio de trabajo
WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./
COPY yarn.lock ./

# Instalar dependencias
RUN npm ci --only=production && npm cache clean --force

# Copiar código fuente
COPY . .

# Construir la aplicación
RUN npm run build

# Stage de producción
FROM nginx:alpine AS production

# Instalar herramientas necesarias
RUN apk add --no-cache curl

# Copiar configuración de nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.conf.d/default.conf /etc/nginx/conf.d/default.conf

# Copiar archivos construidos
COPY --from=builder /app/dist /usr/share/nginx/html

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Cambiar permisos
RUN chown -R nextjs:nodejs /usr/share/nginx/html
RUN chown -R nextjs:nodejs /var/cache/nginx
RUN chown -R nextjs:nodejs /var/log/nginx
RUN chown -R nextjs:nodejs /etc/nginx/conf.d

# Cambiar a usuario no-root
USER nextjs

# Exponer puerto
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Comando de inicio
CMD ["nginx", "-g", "daemon off;"]
```

#### Configuración de Nginx:
```nginx
# nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # Performance optimizations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 16M;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

    # Include server configurations
    include /etc/nginx/conf.d/*.conf;
}

# nginx.conf.d/default.conf
server {
    listen 3000;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Security
    server_tokens off;

    # Rate limiting for API endpoints
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Rate limiting for login
    location /auth/login {
        limit_req zone=login burst=5 nodelay;
        try_files $uri $uri/ /index.html;
    }

    # Static assets with caching
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary Accept-Encoding;
    }

    # HTML files - no cache
    location ~* \.html$ {
        expires -1;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
        add_header Pragma "no-cache";
    }

    # SPA routing
    location / {
        try_files $uri $uri/ /index.html;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }

    # Error pages
    error_page 404 /index.html;
    error_page 500 502 503 504 /50x.html;
}
```

#### Docker Compose para desarrollo:
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Frontend React app
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - REACT_APP_API_URL=http://localhost:8000
      - CHOKIDAR_USEPOLLING=true
    depends_on:
      - backend
    networks:
      - app-network

  # Backend API
  backend:
    image: node:18-alpine
    working_dir: /app
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
    environment:
      - NODE_ENV=development
      - PORT=8000
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      - db
    networks:
      - app-network

  # Base de datos PostgreSQL
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    networks:
      - app-network

  # Redis para cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### 3. Cloud Deployment

#### Deployment en AWS con Terraform:
```hcl
# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "myapp-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      ManagedBy   = "Terraform"
    }
  }
}

# VPC y networking
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "${var.project_name}-vpc"
  cidr = var.vpc_cidr
  
  azs             = var.availability_zones
  private_subnets = var.private_subnet_cidrs
  public_subnets  = var.public_subnet_cidrs
  
  enable_nat_gateway = true
  single_nat_gateway = false
  
  enable_dns_hostnames = true
  enable_dns_support   = true
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "${var.project_name}-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
  
  tags = {
    Name = "${var.project_name}-cluster"
  }
}

# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = "${var.project_name}-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = var.app_cpu
  memory                   = var.app_memory
  execution_role_arn       = aws_iam_role.ecs_execution_role.arn
  task_role_arn            = aws_iam_role.ecs_task_role.arn
  
  container_definitions = jsonencode([
    {
      name  = "${var.project_name}-container"
      image = "${var.ecr_repository_url}:latest"
      
      portMappings = [
        {
          containerPort = 3000
          protocol      = "tcp"
        }
      ]
      
      environment = [
        {
          name  = "NODE_ENV"
          value = "production"
        },
        {
          name  = "REACT_APP_API_URL"
          value = var.api_url
        }
      ]
      
      secrets = [
        {
          name      = "DATABASE_URL"
          valueFrom = aws_secretsmanager_secret.database_url.arn
        }
      ]
      
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          awslogs-group         = aws_cloudwatch_log_group.app_logs.name
          awslogs-region        = var.aws_region
          awslogs-stream-prefix = "ecs"
        }
      }
    }
  ])
  
  tags = {
    Name = "${var.project_name}-task-definition"
  }
}

# ECS Service
resource "aws_ecs_service" "app" {
  name            = "${var.project_name}-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.app_count
  launch_type     = "FARGATE"
  
  network_configuration {
    security_groups  = [aws_security_group.ecs_tasks.id]
    subnets          = module.vpc.private_subnets
    assign_public_ip = false
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "${var.project_name}-container"
    container_port   = 3000
  }
  
  depends_on = [aws_lb_listener.app]
  
  tags = {
    Name = "${var.project_name}-service"
  }
}

# Application Load Balancer
resource "aws_lb" "app" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets
  
  enable_deletion_protection = false
  
  tags = {
    Name = "${var.project_name}-alb"
  }
}

# ALB Target Group
resource "aws_lb_target_group" "app" {
  name        = "${var.project_name}-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = module.vpc.vpc_id
  target_type = "ip"
  
  health_check {
    healthy_threshold   = "3"
    interval            = "30"
    protocol            = "HTTP"
    matcher             = "200"
    timeout             = "3"
    path                = "/health"
    unhealthy_threshold = "2"
  }
  
  tags = {
    Name = "${var.project_name}-target-group"
  }
}

# ALB Listener
resource "aws_lb_listener" "app" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# Security Groups
resource "aws_security_group" "alb" {
  name        = "${var.project_name}-alb-sg"
  description = "Security group for ALB"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    protocol    = "tcp"
    from_port   = 80
    to_port     = 80
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    protocol    = "tcp"
    from_port   = 443
    to_port     = 443
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    protocol    = "-1"
    from_port   = 0
    to_port     = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.project_name}-alb-sg"
  }
}

resource "aws_security_group" "ecs_tasks" {
  name        = "${var.project_name}-ecs-tasks-sg"
  description = "Security group for ECS tasks"
  vpc_id      = module.vpc.vpc_id
  
  ingress {
    protocol        = "tcp"
    from_port       = 3000
    to_port         = 3000
    security_groups = [aws_security_group.alb.id]
  }
  
  egress {
    protocol    = "-1"
    from_port   = 0
    to_port     = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.project_name}-ecs-tasks-sg"
  }
}

# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "app_logs" {
  name              = "/ecs/${var.project_name}"
  retention_in_days = 30
  
  tags = {
    Name = "${var.project_name}-logs"
  }
}

# Secrets Manager
resource "aws_secretsmanager_secret" "database_url" {
  name = "${var.project_name}-database-url"
  
  tags = {
    Name = "${var.project_name}-database-secret"
  }
}

# Variables
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Name of the project"
  type        = string
  default     = "myapp"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
  default     = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}

variable "app_cpu" {
  description = "CPU units for the app"
  type        = number
  default     = 256
}

variable "app_memory" {
  description = "Memory for the app"
  type        = number
  default     = 512
}

variable "app_count" {
  description = "Number of app instances"
  type        = number
  default     = 2
}

variable "ecr_repository_url" {
  description = "ECR repository URL"
  type        = string
}

variable "api_url" {
  description = "API URL"
  type        = string
}

# Outputs
output "alb_dns_name" {
  description = "DNS name of the load balancer"
  value       = aws_lb.app.dns_name
}

output "ecs_cluster_name" {
  description = "Name of the ECS cluster"
  value       = aws_ecs_cluster.main.name
}
```

### 4. Monitoreo y Alertas

#### Configuración de CloudWatch:
```javascript
// src/utils/monitoring.js
import AWS from 'aws-sdk';

// Configurar AWS SDK
AWS.config.update({
  region: process.env.AWS_REGION || 'us-east-1'
});

const cloudwatch = new AWS.CloudWatch();
const logs = new AWS.CloudWatchLogs();

// Métricas personalizadas
export class MetricsCollector {
  constructor(namespace = 'MyApp') {
    this.namespace = namespace;
    this.metrics = [];
  }

  // Métrica de performance
  recordPerformance(metricName, value, unit = 'Milliseconds') {
    this.metrics.push({
      MetricName: metricName,
      Value: value,
      Unit: unit,
      Timestamp: new Date()
    });
  }

  // Métrica de negocio
  recordBusinessMetric(metricName, value, unit = 'Count') {
    this.metrics.push({
      MetricName: metricName,
      Value: value,
      Unit: unit,
      Timestamp: new Date()
    });
  }

  // Métrica de error
  recordError(metricName, count = 1) {
    this.metrics.push({
      MetricName: metricName,
      Value: count,
      Unit: 'Count',
      Timestamp: new Date()
    });
  }

  // Enviar métricas a CloudWatch
  async flush() {
    if (this.metrics.length === 0) return;

    try {
      await cloudwatch.putMetricData({
        Namespace: this.namespace,
        MetricData: this.metrics
      }).promise();

      this.metrics = [];
      console.log('Métricas enviadas a CloudWatch');
    } catch (error) {
      console.error('Error enviando métricas:', error);
    }
  }

  // Enviar métricas automáticamente cada minuto
  startAutoFlush() {
    setInterval(() => {
      this.flush();
    }, 60000);
  }
}

// Logger estructurado
export class StructuredLogger {
  constructor(serviceName = 'MyApp') {
    this.serviceName = serviceName;
  }

  info(message, metadata = {}) {
    this.log('INFO', message, metadata);
  }

  warn(message, metadata = {}) {
    this.log('WARN', message, metadata);
  }

  error(message, error = null, metadata = {}) {
    this.log('ERROR', message, {
      ...metadata,
      error: error ? {
        message: error.message,
        stack: error.stack,
        name: error.name
      } : null
    });
  }

  private log(level, message, metadata = {}) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      service: this.serviceName,
      message,
      ...metadata
    };

    // Log a consola en desarrollo
    if (process.env.NODE_ENV === 'development') {
      console.log(JSON.stringify(logEntry, null, 2));
    }

    // Enviar a CloudWatch Logs en producción
    if (process.env.NODE_ENV === 'production') {
      this.sendToCloudWatch(logEntry);
    }
  }

  private async sendToCloudWatch(logEntry) {
    try {
      await logs.putLogEvents({
        logGroupName: `/myapp/${process.env.NODE_ENV}`,
        logStreamName: `${this.serviceName}-${new Date().toISOString().split('T')[0]}`,
        logEvents: [{
          timestamp: new Date(logEntry.timestamp).getTime(),
          message: JSON.stringify(logEntry)
        }]
      }).promise();
    } catch (error) {
      console.error('Error enviando logs a CloudWatch:', error);
    }
  }
}

// Health checker
export class HealthChecker {
  constructor() {
    this.checks = new Map();
  }

  // Registrar check de salud
  registerCheck(name, checkFunction) {
    this.checks.set(name, checkFunction);
  }

  // Ejecutar todos los checks
  async runChecks() {
    const results = {};
    const promises = [];

    for (const [name, checkFunction] of this.checks) {
      promises.push(
        checkFunction()
          .then(result => {
            results[name] = { status: 'healthy', ...result };
          })
          .catch(error => {
            results[name] = { 
              status: 'unhealthy', 
              error: error.message 
            };
          })
      );
    }

    await Promise.all(promises);
    return results;
  }

  // Verificar si todos los checks están saludables
  async isHealthy() {
    const results = await this.runChecks();
    return Object.values(results).every(result => result.status === 'healthy');
  }
}

// Uso del sistema de monitoreo
const metrics = new MetricsCollector('MyApp');
const logger = new StructuredLogger('Frontend');
const healthChecker = new HealthChecker();

// Registrar checks de salud
healthChecker.registerCheck('database', async () => {
  // Verificar conexión a base de datos
  return { responseTime: 50 };
});

healthChecker.registerCheck('api', async () => {
  // Verificar API externa
  const start = Date.now();
  await fetch('/api/health');
  return { responseTime: Date.now() - start };
});

// Iniciar auto-flush de métricas
metrics.startAutoFlush();

// Exportar para uso en componentes
export { metrics, logger, healthChecker };
```

### 5. Estrategias de Deployment

#### Blue-Green Deployment:
```javascript
// scripts/deploy-blue-green.js
import AWS from 'aws-sdk';
import { execSync } from 'child_process';

const ecs = new AWS.ECS();
const elbv2 = new AWS.ELBV2();

class BlueGreenDeployment {
  constructor(config) {
    this.config = config;
    this.blueTargetGroup = null;
    this.greenTargetGroup = null;
  }

  async deploy() {
    console.log('🚀 Iniciando Blue-Green deployment...');

    try {
      // 1. Crear nueva target group (Green)
      this.greenTargetGroup = await this.createTargetGroup();
      console.log('✅ Target group Green creada');

      // 2. Registrar nueva instancia en Green
      await this.registerTargets(this.greenTargetGroup.TargetGroupArn);
      console.log('✅ Instancias registradas en Green');

      // 3. Ejecutar health checks en Green
      await this.waitForHealthyTargets(this.greenTargetGroup.TargetGroupArn);
      console.log('✅ Health checks pasados en Green');

      // 4. Cambiar tráfico a Green gradualmente
      await this.switchTraffic();
      console.log('✅ Tráfico cambiado a Green');

      // 5. Verificar que Green esté funcionando
      await this.verifyDeployment();
      console.log('✅ Deployment verificado');

      // 6. Limpiar recursos Blue
      await this.cleanupBlue();
      console.log('✅ Recursos Blue limpiados');

      console.log('🎉 Blue-Green deployment completado exitosamente');

    } catch (error) {
      console.error('❌ Error en deployment:', error);
      await this.rollback();
      throw error;
    }
  }

  async createTargetGroup() {
    const params = {
      Name: `${this.config.serviceName}-green-${Date.now()}`,
      Port: this.config.port,
      Protocol: 'HTTP',
      VpcId: this.config.vpcId,
      TargetType: 'ip',
      HealthCheckPath: '/health',
      HealthCheckIntervalSeconds: 30,
      HealthyThresholdCount: 2,
      UnhealthyThresholdCount: 2
    };

    const result = await ecs.createTargetGroup(params).promise();
    return result.TargetGroups[0];
  }

  async registerTargets(targetGroupArn) {
    const targets = this.config.targets.map(target => ({
      Id: target,
      Port: this.config.port
    }));

    await elbv2.registerTargets({
      TargetGroupArn: targetGroupArn,
      Targets: targets
    }).promise();
  }

  async waitForHealthyTargets(targetGroupArn) {
    console.log('⏳ Esperando health checks...');
    
    let attempts = 0;
    const maxAttempts = 20;

    while (attempts < maxAttempts) {
      const health = await elbv2.describeTargetHealth({
        TargetGroupArn: targetGroupArn
      }).promise();

      const healthyTargets = health.TargetHealthDescriptions.filter(
        target => target.TargetHealth.State === 'healthy'
      );

      if (healthyTargets.length === this.config.targets.length) {
        return;
      }

      attempts++;
      await new Promise(resolve => setTimeout(resolve, 30000)); // 30 segundos
    }

    throw new Error('Health checks fallaron después de múltiples intentos');
  }

  async switchTraffic() {
    // Cambiar tráfico gradualmente usando weighted routing
    const params = {
      ListenerArn: this.config.listenerArn,
      DefaultActions: [{
        Type: 'forward',
        TargetGroupArn: this.greenTargetGroup.TargetGroupArn
      }]
    };

    await elbv2.modifyListener(params).promise();
  }

  async verifyDeployment() {
    // Verificar que la nueva versión esté funcionando
    const response = await fetch(`${this.config.loadBalancerUrl}/health`);
    
    if (!response.ok) {
      throw new Error('La nueva versión no está respondiendo correctamente');
    }
  }

  async cleanupBlue() {
    // Eliminar target group Blue anterior
    if (this.config.blueTargetGroupArn) {
      try {
        await elbv2.deleteTargetGroup({
          TargetGroupArn: this.config.blueTargetGroupArn
        }).promise();
      } catch (error) {
        console.warn('No se pudo eliminar target group Blue:', error);
      }
    }
  }

  async rollback() {
    console.log('🔄 Iniciando rollback...');

    try {
      // Cambiar tráfico de vuelta a Blue
      const params = {
        ListenerArn: this.config.listenerArn,
        DefaultActions: [{
          Type: 'forward',
          TargetGroupArn: this.config.blueTargetGroupArn
        }]
      };

      await elbv2.modifyListener(params).promise();

      // Limpiar recursos Green
      if (this.greenTargetGroup) {
        await elbv2.deleteTargetGroup({
          TargetGroupArn: this.greenTargetGroup.TargetGroupArn
        }).promise();
      }

      console.log('✅ Rollback completado');
    } catch (error) {
      console.error('❌ Error en rollback:', error);
    }
  }
}

// Configuración del deployment
const config = {
  serviceName: 'myapp',
  port: 3000,
  vpcId: process.env.VPC_ID,
  targets: process.env.TARGET_IPS.split(','),
  listenerArn: process.env.LISTENER_ARN,
  loadBalancerUrl: process.env.LOAD_BALANCER_URL,
  blueTargetGroupArn: process.env.BLUE_TARGET_GROUP_ARN
};

// Ejecutar deployment
async function main() {
  const deployment = new BlueGreenDeployment(config);
  await deployment.deploy();
}

if (require.main === module) {
  main().catch(console.error);
}

export { BlueGreenDeployment };
```

## 🛠️ Ejercicios Prácticos

### **Ejercicio 1: Pipeline CI/CD Completo**
Configura un pipeline de CI/CD completo con GitHub Actions que incluya testing, building, y deployment.

### **Ejercicio 2: Dockerización de Aplicación**
Crea un Dockerfile optimizado y docker-compose para desarrollo y producción.

### **Ejercicio 3: Deployment en la Nube**
Implementa deployment en AWS, Azure o GCP usando infraestructura como código.

### **Ejercicio 4: Monitoreo y Alertas**
Configura un sistema de monitoreo completo con métricas, logs y alertas.

### **Ejercicio 5: Blue-Green Deployment**
Implementa estrategia de Blue-Green deployment con rollback automático.

### **Ejercicio 6: Gestión de Secretos**
Configura gestión segura de secretos y variables de entorno.

### **Ejercicio 7: CDN y Optimización**
Implementa CDN para assets estáticos y optimización de delivery.

### **Ejercicio 8: Backup y Recovery**
Crea estrategias de backup y disaster recovery para la aplicación.

### **Ejercicio 9: Multi-Environment**
Configura múltiples entornos con deployment automatizado.

### **Ejercicio 10: Security y Compliance**
Implementa escaneo de seguridad y compliance en el pipeline.

## 🎯 Proyecto Integrador: Plataforma DevOps Completa

### **Descripción del Proyecto**
Construye una plataforma DevOps completa que incluya CI/CD, Docker, cloud deployment, monitoreo, y estrategias de producción para una aplicación React empresarial.

### **Requisitos del Proyecto**
- ✅ Pipeline CI/CD completo con GitHub Actions
- ✅ Dockerización optimizada
- ✅ Deployment en la nube con IaC
- ✅ Monitoreo y alertas en tiempo real
- ✅ Estrategias de deployment avanzadas
- ✅ Gestión de secretos y configuración
- ✅ CDN y optimización de assets
- ✅ Backup y disaster recovery
- ✅ Múltiples entornos
- ✅ Documentación técnica completa

## 📝 Autoevaluación

### **Preguntas de Repaso**
1. ¿Qué es CI/CD y cuáles son sus beneficios?
2. ¿Cómo funciona Docker y cuándo usarlo?
3. ¿Qué es infraestructura como código?
4. ¿Cómo se implementa Blue-Green deployment?
5. ¿Qué métricas son importantes monitorear en producción?
6. ¿Cómo se gestionan secretos de manera segura?
7. ¿Cuáles son las mejores prácticas de DevOps?

### **Criterios de Evaluación**
- ✅ Configurar pipelines de CI/CD
- ✅ Crear y gestionar contenedores Docker
- ✅ Implementar deployment en la nube
- ✅ Configurar monitoreo y alertas
- ✅ Implementar estrategias de deployment
- ✅ Gestionar secretos y configuración
- ✅ Configurar CDN y optimización
- ✅ Crear documentación técnica

## 🔑 Conceptos Clave a Recordar

- **CI/CD** automatiza el proceso de desarrollo y deployment
- **Docker** permite empaquetar aplicaciones en contenedores
- **Infraestructura como código** permite gestionar infraestructura con código
- **Blue-Green deployment** reduce downtime durante actualizaciones
- **Monitoreo** es esencial para aplicaciones en producción
- **Gestión de secretos** debe ser segura y automatizada
- **CDN** mejora la performance de assets estáticos
- **DevOps** es una cultura, no solo herramientas

## 🚀 ¡Felicidades! Has Completado el Curso

Has completado exitosamente el curso completo de React de cero a senior. Ahora tienes todas las habilidades necesarias para crear aplicaciones React empresariales de clase mundial.

## 📚 Recursos Adicionales

- [GitHub Actions](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [Terraform](https://www.terraform.io/docs)
- [AWS ECS](https://docs.aws.amazon.com/ecs/)
- [CloudWatch](https://docs.aws.amazon.com/cloudwatch/)

## ✅ Checklist Final de Competencias

- [ ] Configuro pipelines de CI/CD completos
- [ ] Creo y gestiono contenedores Docker
- [ ] Implemento deployment en la nube
- [ ] Configuro monitoreo y alertas de producción
- [ ] Implemento estrategias de deployment avanzadas
- [ ] Gestiono variables de entorno y secretos
- [ ] Configuro CDN y optimización de assets
- [ ] Implemento backup y disaster recovery
- [ ] Gestiono múltiples entornos
- [ ] Completo el proyecto integrador de plataforma DevOps

---

**🎉 ¡FELICIDADES! Has completado el curso completo de React de CERO a SENIOR. Eres ahora un desarrollador React experto con habilidades empresariales.** 🚀

**Próximos pasos recomendados:**
1. **Practica** con proyectos personales
2. **Contribuye** a proyectos open source
3. **Mantente actualizado** con las últimas tecnologías
4. **Comparte conocimiento** con la comunidad
5. **Construye tu portafolio** de proyectos

**¡El futuro del desarrollo web está en tus manos!** 💪
