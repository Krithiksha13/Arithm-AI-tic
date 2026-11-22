# Arithm-AI-tic
An AI-powered application that creates a financial survival plan based on unpredictable income. Features: Predicts future income based on past earning patterns, Calculates how much money should be reserved for essential needs, warns users when expense exceeds available money,estimates days to manage before next income.
AI used:Chatgpt
demo link: https://youtu.be/PVXCD_LLhXQ?si=fy0JOb7s3rkGR1GR
Code:
Step 1:save the below code as setup_kaasukavalan.py

from pathlib import Path

BASE = Path("kaasukavalan_app")

REQUIREMENTS_TXT = """fastapi>=0.115.0
uvicorn>=0.32.0
pydantic>=2.9.0
google-genai>=1.0.0
httpx>=0.27.0
"""

BACKEND_MAIN_PY = r"""from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import FileResponse
from pydantic import BaseModel, Field
from typing import List, Optional, Literal
from pathlib import Path
import statistics
import re
import os

app = FastAPI(title="KaasuKavalan - Financial Survival AI", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

FRONTEND_DIR = Path(__file__).resolve().parent.parent / "frontend"

# --- DATA MODELS ---

class EssentialItem(BaseModel):
    name: str
    amount: float
    priority: Literal[1, 2, 3]
    due_in_days: int
    can_split_minimum: Optional[float] = None

class VendorSnapshot(BaseModel):
    daily_incomes: List[float] = Field(..., description="Recent daily earnings, e.g. [800, 300, 0]")
    available_balance: float
    upcoming_essentials: List[EssentialItem]
    rain_forecast_next_3_days: bool = False
    daily_living_baseline: float = 150.0

class FutureSimRequest(BaseModel):
    available_balance: float
    upcoming_essentials: List[EssentialItem]
    zero_income_days: int = 5
    daily_living_baseline: float = 150.0

class VoiceQueryRequest(BaseModel):
    transcript: str
    available_balance: float
    upcoming_essentials: List[EssentialItem]
    language: Literal["tanglish", "tamil", "english"] = "tanglish"

class ScamCheckRequest(BaseModel):
    claimed_amount: float
    ocr_screenshot_text: Optional[str] = None
    bank_sms_received: bool = False
    bank_sms_text: Optional[str] = None
    suspicious_link_or_sms: Optional[str] = None

class BenefitQuery(BaseModel):
    occupation: str = "street_vendor"
    sta