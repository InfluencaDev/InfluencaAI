import tweepy
import openai
import time

# Twitter API settings
TWITTER_API_KEY = "your_twitter_api_key"
TWITTER_API_SECRET = "your_twitter_api_secret"
TWITTER_ACCESS_TOKEN = "your_twitter_access_token"
TWITTER_ACCESS_SECRET = "your_twitter_access_secret"

# OpenAI settings
OPENAI_API_KEY = "your_openai_api_key"
openai.api_key = OPENAI_API_KEY

# Authenticate with Twitter
auth = tweepy.OAuthHandler(TWITTER_API_KEY, TWITTER_API_SECRET)
auth.set_access_token(TWITTER_ACCESS_TOKEN, TWITTER_ACCESS_SECRET)
api = tweepy.API(auth)

# Function to generate a response using OpenAI
def generate_response(prompt):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}]
    )
    return response["choices"][0]["message"]["content"].strip()

# Function to handle mentions
def reply_to_mentions():
    mentions = api.mentions_timeline(count=5)  # Get the last 5 mentions
    for mention in mentions:
        tweet_text = mention.text.replace("@your_bot_username", "").strip()
        response = generate_response(tweet_text)
        api.update_status(f"@{mention.user.screen_name} {response}", in_reply_to_status_id=mention.id)
        time.sleep(5)  # Anti-spam delay

# Run in a loop
while True:
    try:
        reply_to_mentions()
        time.sleep(60)  # Check for mentions every minute
    except Exception as e:
        print(f"Error: {e}")
        time.sleep(60)
