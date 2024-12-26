
import streamlit as st
import datetime
from streamlit_folium import st_folium
import folium
from urllib.request import urlopen
import json

def fetch_weather():
    # Fetch the sample URL
    base_url = "https://samples.openweathermap.org/"
    res = urlopen(base_url).read()
    json_data = json.loads(res)
    sample_url = json_data["products"]["current_weather"]["samples"][0]

    # Fetch weather data
    res = urlopen(sample_url).read()
    return json.loads(res)

def display_weather(data):
    st.title(f"Weather in {data['name']}, {data['sys']['country']}")

    # Map
    m = folium.Map(location=[data['coord']['lat'], data['coord']['lon']], zoom_start=10)
    folium.Marker(
        [data['coord']['lat'], data['coord']['lon']],
        popup=folium.Popup(f"<b>City:</b> {data['name']}<br><b>Condition:</b> {data['weather'][0]['description'].capitalize()}<br><b>Temperature:</b> {data['main']['temp'] - 273.15:.2f} °C", max_width=250)
    ).add_to(m)
    st_folium(m, width=700, height=500)

    # Sidebar
    with st.sidebar:
        st.subheader("Weather Details")

        # Main Weather Description
        weather = data['weather'][0]
        st.write(f"**Condition**: {weather['main']}")
        st.write(f"{weather['description'].capitalize()}")
        
        # Temperature in Celsius
        temp = data['main']
        temp_celsius = lambda k: k - 273.15
        st.metric("Temperature", f"{temp_celsius(temp['temp']):.2f} °C", delta=f"Min: {temp_celsius(temp['temp_min']):.2f} °C | Max: {temp_celsius(temp['temp_max']):.2f} °C")
        
        # Additional details
        st.write("**Details**:")
        st.write(f"- Pressure: {temp['pressure']} hPa")
        st.write(f"- Humidity: {temp['humidity']}%")

        # Visibility and Wind
        st.write(f"**Visibility**: {data['visibility']} meters")
        wind = data['wind']
        st.write(f"**Wind**: {wind['speed']} m/s at {wind['deg']}°")

        # Cloud Cover
        st.write(f"**Cloud Cover**: {data['clouds']['all']}%")

        # Sunrise and Sunset
        sunrise = datetime.datetime.fromtimestamp(data['sys']['sunrise'])
        sunset = datetime.datetime.fromtimestamp(data['sys']['sunset'])
        st.write(f"**Sunrise**: {sunrise.strftime('%H:%M:%S')} UTC")
        st.write(f"**Sunset**: {sunset.strftime('%H:%M:%S')} UTC")

# Fetch and display the weather data
data = fetch_weather()
display_weather(data)