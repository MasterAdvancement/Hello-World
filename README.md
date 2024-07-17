import ui
from datetime import datetime, timedelta
import requests
import random
import colorsys

NEWS_API_KEY = "be9fe5472c42460fac0e939382838dbb"  # Replace with your actual News API key

def get_news_headlines():
    url = f"https://newsapi.org/v2/top-headlines?country=us&apiKey={NEWS_API_KEY}"
    try:
        response = requests.get(url)
        response.raise_for_status()
        news_data = response.json()
        headlines = [article['title'] for article in news_data['articles'][:5]]  # Get top 5 headlines
        return headlines
    except Exception as e:
        print(f"Error fetching news: {e}")
        return ["Unable to fetch news headlines"]

def get_background_color():
    current_hour = datetime.now().hour
    # Normalize the hour to a value between 0 and 1
    normalized_hour = (current_hour % 24) / 24.0
    
    # Use HSV color model for smooth transitions
    # Hue: 0.6 (blue) at midnight, transitioning to 0.1 (orange) at noon
    hue = 0.6 - (normalized_hour * 0.5) if normalized_hour < 0.5 else 0.1 + ((normalized_hour - 0.5) * 1.0)
    # Saturation: 30% to 50%
    saturation = 0.3 + (normalized_hour * 0.2)
    # Value (brightness): 20% at night, 60% during the day
    value = 0.2 + (normalized_hour * 0.4) if normalized_hour < 0.5 else 0.6 - ((normalized_hour - 0.5) * 0.4)
    
    # Convert HSV to RGB
    rgb = colorsys.hsv_to_rgb(hue, saturation, value)
    
    # Convert RGB values to hexadecimal color code
    return '#{:02x}{:02x}{:02x}'.format(int(rgb[0]*255), int(rgb[1]*255), int(rgb[2]*255))

weather_icons = {
    'clear sky': '☀️',
    'few clouds': '🌤️',
    'scattered clouds': '☁️',
    'broken clouds': '☁️',
    'shower rain': '🌧️',
    'rain': '🌧️',
    'thunderstorm': '⛈️',
    'snow': '❄️',
    'mist': '🌫️'
}

locations = ["New York", "Los Angeles", "London", "Tokyo", "Sydney"]
time_zones = ["UTC", "US/Pacific", "Europe/London", "Asia/Tokyo", "Australia/Sydney"]
location_index = 0
time_zone_index = 0

# List of daily affirmations
affirmations = [
    "You are capable of amazing things.",
    "Believe in yourself and all that you are.",
    "The best is yet to come.",
    "You are stronger than you think.",
    "Every day is a second chance."
]

def get_weather(location):
    api_key = "6bc4f918fa383b7d00ad994b205f1333"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={location}&appid={api_key}&units=metric"
    try:
        response = requests.get(url)
        response.raise_for_status()
        weather_data = response.json()
        temperature = weather_data['main']['temp']
        weather = weather_data['weather'][0]['description']
        icon = weather_icons.get(weather.lower(), '🌡️')  # Default to thermometer if no match
        return f"{icon} {temperature:.1f}°C, {weather.capitalize()}"
    except Exception as e:
        print(f"Error fetching weather: {e}")
        return "Weather data unavailable"
        
def show_affirmation():
    return random.choice(affirmations)

def update_display(sender):
    global location_index, time_zone_index
    current_time = datetime.now().strftime("%H:%M:%S")
    current_date = datetime.now().strftime("%A, %B %d, %Y")
    sender['time_label'].text = current_time
    sender['date_label'].text = current_date
    
    # Update background color
    sender.background_color = get_background_color()
    
    # Update weather every hour
    if not hasattr(sender, 'last_weather_update') or (datetime.now() - sender.last_weather_update) >= timedelta(hours=1):
        location = locations[location_index]
        sender['weather_label'].text = get_weather(location)
        sender.last_weather_update = datetime.now()
    
    # Update news headlines every hour
    if not hasattr(sender, 'last_news_update') or (datetime.now() - sender.last_news_update) >= timedelta(hours=1):
        headlines = get_news_headlines()
        sender['news_label'].text = "\n\n".join(headlines)
        sender.last_news_update = datetime.now()
    
    # Update affirmation every day
    if not hasattr(sender, 'last_affirmation_update') or (datetime.now().date() - sender.last_affirmation_update.date()).days >= 1:
        sender['affirmation_label'].text = show_affirmation()
        sender.last_affirmation_update = datetime.now()
    
    ui.delay(lambda: update_display(sender), 1)

def cycle_location(sender):
    global location_index
    location_index = (location_index + 1) % len(locations)
    sender.last_weather_update = datetime.now() - timedelta(hours=1)  # Force update

def toggle_news(sender):
    news_label = sender.superview['news_label']  # Retrieve the news_label from the superview
    if news_label.hidden:
        news_label.hidden = False
    else:
        news_label.hidden = True

def setup_ui():
    screen_width, screen_height = ui.get_screen_size()
    v = ui.View(frame=(0, 0, screen_width, screen_height))
    v.background_color = get_background_color()
    
    time_label = ui.Label()
    time_label.name = 'time_label'
    time_label.font = ('Helvetica', int(screen_height * 0.1))
    time_label.text_color = '#FFFFFF'
    time_label.alignment = ui.ALIGN_CENTER
    time_label.frame = (0, screen_height * 0.1, screen_width, screen_height * 0.15)
    time_label.touch_enabled = True
    time_label.action = cycle_location
    v.add_subview(time_label)

    date_label = ui.Label()
    date_label.name = 'date_label'
    date_label.font = ('Helvetica', int(screen_height * 0.03))
    date_label.text_color = '#FFFFFF'
    date_label.alignment = ui.ALIGN_CENTER
    date_label.frame = (0, screen_height * 0.25, screen_width, screen_height * 0.05)
    v.add_subview(date_label)

    weather_label = ui.Label()
    weather_label.name = 'weather_label'
    weather_label.font = ('Helvetica', int(screen_height * 0.05))
    weather_label.text_color = '#FFFFFF'
    weather_label.alignment = ui.ALIGN_CENTER
    weather_label.frame = (0, screen_height * 0.3, screen_width, screen_height * 0.1)
    weather_label.touch_enabled = True
    weather_label.action = cycle_location
    v.add_subview(weather_label)

    news_bubble = ui.Button()
    news_bubble.name = 'news_bubble'
    news_bubble.font = ('Helvetica', int(screen_height * 0.03))
    news_bubble.title = 'News'
    news_bubble.tint_color = '#FFFFFF'
    news_bubble.background_color = '#444444'
    news_bubble.corner_radius = int(screen_height * 0.05 / 2)  # Adjusted for int type
    news_bubble.frame = (screen_width * 0.4, screen_height * 0.42, screen_width * 0.2, screen_height * 0.05)
    news_bubble.action = toggle_news
    v.add_subview(news_bubble)

    news_label = ui.Label()
    news_label.name = 'news_label'
    news_label.font = ('Helvetica', int(screen_height * 0.02))
    news_label.text_color = '#FFFFFF'
    news_label.alignment = ui.ALIGN_LEFT
    news_label.number_of_lines = 0
    news_label.frame = (screen_width * 0.05, screen_height * 0.5, screen_width * 0.9, screen_height * 0.3)
    news_label.hidden = True
    v.add_subview(news_label)

    affirmation_label = ui.Label()
    affirmation_label.name = 'affirmation_label'
    affirmation_label.font = ('Helvetica', int(screen_height * 0.03))
    affirmation_label.text_color = '#FFFFFF'
    affirmation_label.alignment = ui.ALIGN_CENTER
    affirmation_label.number_of_lines = 0
    affirmation_label.frame = (screen_width * 0.1, screen_height * 0.9, screen_width * 0.8, screen_height * 0.2)
    v.add_subview(affirmation_label)

    update_display(v)
    return v

if __name__ == '__main__':
    main_view = setup_ui()
    main_view.present('fullscreen')