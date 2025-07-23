# Bài Thực Hành GitHub Actions

## Bài 1: Tự động kiểm tra Pull Request

**Mục tiêu:** Thiết lập CI workflow chạy test khi PR được tạo.

### Hướng dẫn:

1. Tạo project Node.js đơn giản:
   ```bash
   npm init -y
   npm install --save-dev jest
   ```
2. Thêm file `sum.js`:
   ```js
   function sum(a, b) {
     return a + b;
   }
   module.exports = sum;
   ```
3. Thêm file test `sum.test.js`:
   ```js
   const sum = require('./sum');
   test('adds 1 + 2 to equal 3', () => {
     expect(sum(1, 2)).toBe(3);
   });
   ```
4. Cập nhật `package.json`:
   ```json
   "scripts": {
     "test": "jest"
   }
   ```
5. Tạo file `.github/workflows/pr-check.yml`:
   ```yaml
   name: PR Check
   on: [pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Setup Node
           uses: actions/setup-node@v3
           with:
             node-version: '20'
         - run: npm install
         - run: npm test
   ```
6. Commit và push lên GitHub. Tạo PR để xem Action chạy.

---

## Bài 2: Kiểm tra bảo mật với Trivy

**Mục tiêu:** Scan Docker image phát hiện lỗi bảo mật

### Hướng dẫn:

1. Viết Dockerfile cho app Python:
   ```Dockerfile
   FROM python:3.10
   COPY . /app
   WORKDIR /app
   RUN pip install flask
   CMD ["python", "app.py"]
   ```
2. Tạo app.py:
   ```python
   from flask import Flask
   app = Flask(__name__)
   @app.route('/')
   def home():
       return 'Hello World'
   if __name__ == '__main__':
       app.run()
   ```
3. Tạo workflow `.github/workflows/trivy.yml`:
   ```yaml
   name: Trivy Scan
   on: [push]
   jobs:
     scan:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Build Docker Image
           run: docker build -t myapp .
         - name: Trivy Scan
           uses: aquasecurity/trivy-action@master
           with:
             image-ref: 'myapp'
   ```

---

## Bài 3: Build & Push Docker Image

**Mục tiêu:** Đẩy Docker image lên Docker Hub

### Hướng dẫn:

1. Tạo Dockerfile giống Bài 2
2. Tạo DockerHub account và lấy access token
3. Thêm secrets `DOCKER_USERNAME`, `DOCKER_PASSWORD` vào repo GitHub
4. Tạo workflow `.github/workflows/docker.yml`:
   ```yaml
   name: Build and Push Docker
   on: [push]
   jobs:
     docker:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Login DockerHub
           run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
         - run: docker build -t ${{ secrets.DOCKER_USERNAME }}/myapp .
         - run: docker push ${{ secrets.DOCKER_USERNAME }}/myapp
   ```

---
