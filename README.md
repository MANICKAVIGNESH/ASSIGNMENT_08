


import pandas as pd
import streamlit as st
import re
import speech_recognition as sr

# Load the dataset
data_frame = pd.read_csv("C:/Users/manic/Downloads/Train_details_22122017 (1).csv", low_memory=False)


# Replace "JN" with "JN." in the specified columns
data_frame['Station Name'] = data_frame['Station Name'].str.replace('JN', 'JN.')
data_frame['Destination Station Name'] = data_frame['Destination Station Name'].str.replace('JN', 'JN.')


# Function to extract 'from' and 'to' stations from user input
def extract_stations(input_string):
    # Split the input string into words
    words = input_string.split()

    # Convert words to lowercase
    words_lower = [word.lower() for word in words]

    # Find the indices of "from" and "to" in the words list
    from_index = words_lower.index('from') if 'from' in words_lower else None
    to_index = words_lower.index('to') if 'to' in words_lower else None

    # If "from" and "to" both exist, extract stations between them
    if from_index is not None and to_index is not None:
        # Extract the station names
        from_station = ' '.join(words[from_index + 1:to_index])
        to_station = ' '.join(words[to_index + 1:])
        
        # Check if "JUNCTION" exists in the station names and replace it with "JN."
        from_station = from_station.replace("junction", "JN.")
        to_station = to_station.replace("junction", "JN.")

        # Capitalize station names
        from_station = from_station.capitalize()
        to_station = to_station.capitalize()

        return from_station, to_station

    # If "from" or "to" is missing, return None for both stations
    return None, None



# Function to find and print train timetable between 'from' and 'to' stations
def find_trains_between_stations(from_station, to_station):
    # Filter rows where 'Station Name' matches the input station names for both 'from' and 'to'
    from_station = from_station.upper()
    to_station = to_station.upper()
    
    from_rows = data_frame[data_frame['Station Name'].str.upper().str.contains(from_station, na=False)]
    to_rows = data_frame[data_frame['Destination Station Name'].str.upper().str.contains(to_station, na=False)]
    
    # Merge the two DataFrames to find common 'Train No' between 'from' and 'to' stations
    common_trains = pd.merge(from_rows, to_rows, on='Train No', how='inner')

    # Drop duplicates based on Train No
    common_trains = common_trains.drop_duplicates(subset='Train No')

    # Print the train timetable between 'from' and 'to' stations
    if not common_trains.empty:
        st.write(f"FROM: {from_station}, TO: {to_station}\n")
        st.write(common_trains)
    else:
        st.write(f"No trains found from '{from_station}' to '{to_station}'")

# Streamlit UI
st.title(":orange[MyTrain Chatbot]")
st.header("Instruction")
st.caption("Say sentence as per below example")
st.caption("Example: Can I know the available trains from Coimbatore to Chennai Central")

user_input = st.text_input("Enter your query:")

# Add support for voice input
if st.button('Find Trains'):
    if user_input:
        if re.search(r'\bfrom\b', user_input, re.IGNORECASE) and re.search(r'\bto\b', user_input, re.IGNORECASE):
            from_station, to_station = extract_stations(user_input)
            if from_station and to_station:
                st.write(f"FROM: {from_station}, TO: {to_station}\n")
                find_trains_between_stations(from_station, to_station)
            else:
                st.write("Please provide both 'from' and 'to' stations in your query.")
        else:
            st.write("Please provide both 'from' and 'to' stations in your query.")
    else:
        st.write("Please enter your query.")

# Voice Input functionality
if st.button('Speak'):
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        st.write("Speak Anything:")
        audio = recognizer.listen(source)
        try:
            user_input = recognizer.recognize_google(audio)
            st.write("You said: {}".format(user_input))
        except:
            st.write("Sorry could not recognize your voice")
