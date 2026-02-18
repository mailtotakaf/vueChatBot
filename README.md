# vueChatBot

---
## Setup Instructions

### 1. Dify Setup (dify.ai)
Log in to [Dify.ai](https://dify.ai/) and prepare your environment:

- **Create an App:** Select "Create from Scratch" → "Chatbot".
- **Generate API Key:** Go to **API Access** in the left menu, click "Create API Key," and save it somewhere safe.
- **Check API Base URL:** On the same page, find your API Base URL (e.g., `https://api.dify.ai/v1`).

### 2. Prepare Vue for API Requests
To make API calls from the browser, install **axios** by running the following command in your terminal:

```bash
npm install axios
```

### 3. Configure Environment Variables
Create a file named .env in your project's root folder and add your API key:
```
VITE_DIFY_API_KEY=<your_api_key_here>
```

===================================================
### CloudFormation デプロイ
```
aws cloudformation create-stack --stack-name vue-chatbot-infra --template-body file://deploy.yml --capabilities CAPABILITY_IAM
```
===================================================

### ビルド（distフォルダ内に生成される）
```
npm run build
```

### ファイルのアップロード
dist フォルダの中身をS3に上げる。
```
aws s3 sync dist/ s3://vue-chatbot-infra-hosting-bucket/
```

### CloudFrontキャッシュクリア
```
aws cloudfront create-invalidation \
  --distribution-id 【ディストリビューションID】 \
  --paths "/*"
```