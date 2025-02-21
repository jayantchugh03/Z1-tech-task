from fastapi import FastAPI, File, UploadFile, HTTPException
from fastapi.responses import JSONResponse
from PIL import Image
import io
import tweepy
import os
from dotenv import load_dotenv

app = FastAPI()

# Load environment variables
load_dotenv()

# Define target sizes
TARGET_SIZES = [(300, 250), (728, 90), (160, 600), (300, 600)]

# Twitter API Credentials (loaded from .env file)
TWITTER_API_KEY = os.getenv("TWITTER_API_KEY")
TWITTER_API_SECRET = os.getenv("TWITTER_API_SECRET")
TWITTER_ACCESS_TOKEN = os.getenv("TWITTER_ACCESS_TOKEN")
TWITTER_ACCESS_SECRET = os.getenv("TWITTER_ACCESS_SECRET")

auth = tweepy.OAuth1UserHandler(
    TWITTER_API_KEY, TWITTER_API_SECRET, TWITTER_ACCESS_TOKEN, TWITTER_ACCESS_SECRET
)
api = tweepy.API(auth)

@app.post("/upload/")
async def upload_image(file: UploadFile = File(...)):
    try:
        image = Image.open(io.BytesIO(await file.read()))
        resized_images = []
        
        for size in TARGET_SIZES:
            img_resized = image.resize(size, Image.ANTIALIAS)
            img_bytes = io.BytesIO()
            img_resized.save(img_bytes, format="PNG")
            img_bytes.seek(0)
            resized_images.append(img_bytes)
        
        media_ids = []
        for img in resized_images:
            res = api.media_upload(filename="resized.png", file=img)
            media_ids.append(res.media_id_string)
        
        api.update_status(status="Here are your resized images!", media_ids=media_ids)
        
        return JSONResponse(content={"message": "Images uploaded successfully!"})
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
