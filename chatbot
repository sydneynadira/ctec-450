from botbuilder.core import ActivityHandler, TurnContext, BotFrameworkAdapter, BotFrameworkAdapterSettings
from botbuilder.schema import Activity
from aiohttp import web
import os

# ✅ Correct adapter initialization
settings = BotFrameworkAdapterSettings(
    app_id=os.getenv("MicrosoftAppId"),
    app_password=os.getenv("MicrosoftAppPassword")
)
adapter = BotFrameworkAdapter(settings)

# ✅ Store user conversation state
USER_STATE = {}

# ✅ Updated Travel Recommendations Dictionary (Includes City, Historic, Adventure & U.S. Locations)
travel_recommendations = {
    "low": {
        "beach": [
            {"location": "Myrtle Beach, SC, USA", "flight": "$200", "stay": "$150", "activities": "$100",
             "preferences": ["swimming", "sightseeing"],
             "description": "Affordable beach destination with boardwalk entertainment."}
        ],
        "city": [
            {"location": "Chicago, IL, USA", "flight": "$300", "stay": "$250", "activities": "$150",
             "preferences": ["shopping", "sightseeing"],
             "description": "Famous for deep-dish pizza, museums, and shopping."}
        ],
        "historic": [
            {"location": "Washington, D.C., USA", "flight": "$300", "stay": "$200", "activities": "$150",
             "preferences": ["sightseeing"], "description": "Visit the U.S. Capitol, monuments, and free museums."}
        ]
    },
    "medium": {
        "city": [
            {"location": "New Orleans, LA, USA", "flight": "$500", "stay": "$600", "activities": "$300",
             "preferences": ["shopping", "nightlife"],
             "description": "Live jazz, French Quarter charm, and vibrant nightlife."}
        ],
        "historic": [
            {"location": "Boston, MA, USA", "flight": "$500", "stay": "$600", "activities": "$300",
             "preferences": ["sightseeing", "shopping"],
             "description": "Walk the Freedom Trail and explore colonial history."}
        ]
    },
    "luxury": {
        "city": [
            {"location": "Los Angeles, CA, USA", "flight": "$800", "stay": "$1200", "activities": "$1000",
             "preferences": ["shopping", "sightseeing"],
             "description": "Hollywood, Rodeo Drive, and exclusive entertainment."}
        ],
        "historic": [
            {"location": "Paris, France", "flight": "$1500", "stay": "$2000", "activities": "$1500",
             "preferences": ["sightseeing", "shopping"],
             "description": "World-class museums, fine dining, and a rich history."}
        ]
    }
}


class TravelBot(ActivityHandler):
    async def on_message_activity(self, turn_context: TurnContext):
        """Handles conversation logic for the bot."""

        user_message = turn_context.activity.text.lower().strip()
        conversation_id = turn_context.activity.conversation.id

        if conversation_id not in USER_STATE:
            USER_STATE[conversation_id] = {}

        user_state = USER_STATE[conversation_id]

        # ✅ Greeting message
        if "hi" in user_message or "hello" in user_message:
            response = "Hi, welcome to TravelBuddy! 🌍✈️\nI help you plan the best vacation based on your budget and interests! 🎒💼\nWould you like to get started? (yes/no)"
            user_state.clear()  # Reset conversation state

        # ✅ Ask for budget
        elif "yes" in user_message:
            response = "Soooo… what's your budget? 👀💰 (Enter any amount between $500 and $5000)"

        # ✅ Process budget input
        elif user_message.isdigit():
            budget = int(user_message)
            category = "luxury" if budget == 5000 else "medium" if budget >= 1500 else "low"
            user_state["budget_category"] = category
            response = (
                f"Got it! Your budget is ${budget} 💰\n"
                "What type of vacation are you looking for? 🌴🏙️🏛️🚀💎\n"
                "(Choose: Beach, City, Historic, Adventure, Luxury)"
            )

        # ✅ Process vacation type (Fixes "historic" issue)
        elif user_message in ["beach", "city", "historic", "adventure", "luxury"]:
            if user_state.get("budget_category") and user_message in travel_recommendations[
                user_state["budget_category"]]:
                user_state["vacation_type"] = user_message
                response = "What activities do you enjoy? 🎭🎒💃\n(Choose: Swimming, Shopping, Sightseeing, Hiking, Nightlife)"
            else:
                response = "Oops! That vacation type isn't available for your budget. Try again!"

        # ✅ Process activity preference (Fixes activity issues)
        elif user_message in ["swimming", "shopping", "sightseeing", "hiking", "nightlife"]:
            user_state["activity_preference"] = user_message
            category = user_state.get("budget_category", "")
            vacation_type = user_state.get("vacation_type", "")

            if category and vacation_type in travel_recommendations.get(category, {}):
                recommendations = [
                    rec for rec in travel_recommendations[category][vacation_type]
                    if user_message in rec["preferences"]
                ]

                if recommendations:
                    response = "Here are some great places for you:\n"
                    for rec in recommendations:
                        response += f"- {rec['location']} 🌎\n  Flight: {rec['flight']} ✈️\n  Stay: {rec['stay']} 🏨\n  Activities: {rec['activities']} 🎉\n  Why? {rec['description']}\n\n"
                    response += "Would you like to start over and try another budget or vacation type? (yes/no)"
                else:
                    response = "Sorry, no matching recommendations. Try another activity or vacation type."
            else:
                response = "Something went wrong! Try another vacation type."

        elif "no" in user_message:
            response = "Okay! Safe travels! ✈️🌍"

        else:
            response = "I didn't quite get that. Can you try again?"

        await turn_context.send_activity(response)


# ✅ API ROUTING & BOT INITIALIZATION
async def handle(request):
    body = await request.json()
    activity = Activity().deserialize(body)

    async def call_bot(turn_context):
        await bot.on_turn(turn_context)

    auth_header = request.headers.get("Authorization", "")
    await adapter.process_activity(activity, auth_header, call_bot)

    return web.json_response({"status": "success"}, status=200)


# ✅ Initialize bot and web app
app = web.Application()
bot = TravelBot()

app.router.add_get("/", lambda request: web.Response(text="Bot is running!"))
app.router.add_post("/api/messages", handle)

# ✅ RUN THE BOT
if __name__ == "__main__":
    print("Bot is starting... 🚀")
    web.run_app(app, host="127.0.0.1", port=3979)
