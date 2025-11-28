import os
import json
import logging
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

# 导入 Sheets 库
import gspread
import requests
from google.oauth2.service_account import Credentials

# --- 配置和初始化 ---
app = FastAPI()
logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')

# 从 Vercel 环境变量中获取配置
# 注意：GOOGLE_SERVICE_ACCOUNT_JSON 是一个 JSON 字符串，而不是文件路径
SERVICE_ACCOUNT_JSON = os.environ.get("GOOGLE_SERVICE_ACCOUNT_JSON")
SPREADSHEET_ID = os.environ.get("SPREADSHEET_ID")
GEMINI_PROXY_URL = os.environ.get("GEMINI_PROXY_URL")
WEBHOOK_SECRET = os.environ.get("WEBHOOK_SECRET") # 用于安全验证

# 流程常量
STATUS_TO_PROCESS = "Draft" 
STATUS_PROCESSING = "Processing"
STATUS_DONE = "Done"

# Webhook 输入模型（可根据实际触发数据调整）
class WebhookPayload(BaseModel):
    sheet_id: str = None
    secret: str = None

# --- 核心连接函数 ---
def get_sheets_client():
    if not SERVICE_ACCOUNT_JSON:
        raise HTTPException(status_code=500, detail="Service Account JSON not configured.")
        
    try:
        # 将 JSON 字符串转换为 Python 字典
        info = json.loads(SERVICE_ACCOUNT_JSON)
        scopes = ['https://www.googleapis.com/auth/spreadsheets', 'https://www.googleapis.com/auth/drive']
        
        # 使用 from_service_account_info 直接认证
        creds = Credentials.from_service_account_info(info, scopes=scopes)
        client = gspread.authorize(creds)
        
        return client
    except Exception as e:
        logging.error(f"Authentication Error: {e}")
        raise HTTPException(status_code=500, detail="Failed to authenticate with Google Sheets.")

# (这里省略了 Colab 中 process_sheet 的核心逻辑，但你需要将其全部复制到这个文件内，
# 并确保它使用 get_sheets_client() 返回的 client 对象来操作表格。)

# 示例：简化的处理逻辑（你需要替换成你自己的完整逻辑）
def process_automation_logic(client, sheet_id):
    try:
        spreadsheet = client.open_by_key(sheet_id)
        sheet = spreadsheet.sheet1
        # ... 你的原有逻辑：读取 Draft 行，调用 Gemini，更新 Status/Content ...
        return f"Sheet {sheet_id} processed successfully."
    except Exception as e:
        logging.error(f"Processing Error: {e}")
        return f"Processing failed: {e}"


# --- FastAPI 暴露接口 ---
@app.post("/api")
def webhook_trigger(payload: WebhookPayload):
    # 1. 安全验证 (防止恶意触发)
    if WEBHOOK_SECRET and payload.secret != WEBHOOK_SECRET:
        raise HTTPException(status_code=403, detail="Invalid Webhook Secret.")

    logging.info(f"Webhook triggered for Sheet ID: {payload.sheet_id}")
    
    try:
        # 2. 获取 Sheets 客户端
        client = get_sheets_client()
        
        # 3. 执行核心自动化逻辑
        result = process_automation_logic(client, payload.sheet_id or SPREADSHEET_ID)
        
        return {"status": "success", "message": result}
        
    except HTTPException as e:
        return {"status": "error", "message": e.detail}
    except Exception as e:
        return {"status": "error", "message": f"An unexpected error occurred: {str(e)}"}
