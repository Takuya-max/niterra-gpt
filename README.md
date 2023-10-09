#環境変数の設定
PROJECT_ID= dx-generativeai-dev
REGION=asia-northeast1
AR_REPO=niterra-gpt
SERVICE_NAME=niterra-gpt
SA_NAME=niterra-gpt
  
# プロジェクト設定の変更
gcloud config set project ${PROJECT_ID}

# API の有効化
gcloud services enable --project=$PROJECT_ID  run.googleapis.com \
  artifactregistry.googleapis.com \
  cloudbuild.googleapis.com \
  compute.googleapis.com \
  aiplatform.googleapis.com \
  iap.googleapis.com
  
# サービスアカウント作成
gcloud iam service-accounts create $SA_NAME
  
# サービスアカウントへ権限付与
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# Artifacts repositories 作成
gcloud artifacts repositories create $AR_REPO \
  --location=$REGION \
  --repository-format=Docker \
  --project=$PROJECT_ID
  
# イメージの作成＆更新
gcloud builds submit --tag $REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME \
  --project=$PROJECT_ID
  
# Cloud Run デプロイ
gcloud run deploy $SERVICE_NAME --port 7860 \
  --image $REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME \
  --no-allow-unauthenticated \
  --service-account=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com \
  --ingress=internal-and-cloud-load-balancing \
  --region=$REGION \
  --set-env-vars=PROJECT_ID=$PROJECT_ID,LOCATION=$REGION \
  --project=$PROJECT_ID
