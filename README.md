# Spotify-Recommendation-System
Please visit the following link to access the demo version: [Spotify-Recommendation-System](https://longliveruby-spotify-recommendation-system.hf.space/)



https://user-images.githubusercontent.com/107134115/201241072-06681109-72ad-4416-b5f0-35322646dc1e.mp4




## Description
The goal of this project is to create a recommendation system that would allow users to discover music based on a given playlist or song that they already enjoy. This project begins with data collection and a self-growing dataset to ensure that the model will work well in the future and continues through model deployment.

## Data
For this project, I'm using the Million Playlist Dataset, which, as its name implies, consists of one million playlists.
contains a number of songs, and some metadata is included as well, such as the name of the playlist, duration, number of songs, number of artists, etc.

It is created by sampling playlists from the billions of playlists that Spotify users have created over the years. 
Playlists that meet the following criteria were selected at random:
- Created by a user that resides in the United States and is at least 13 years old
- Was a public playlist at the time the MPD was generated
- Contains at least 5 tracks
- Contains no more than 250 tracks
- Contains at least 3 unique artists
- Contains at least 2 unique albums
- Has no local tracks (local tracks are non-Spotify tracks that a user has on their local device
- Has at least one follower (not including the creator
- Was created after January 1, 2010 and before December 1, 2017
- Does not have an offensive title
- Does not have an adult-oriented title if the playlist was created by a user under 18 years of age

Check out the dataset [here](https://www.aicrowd.com/challenges/spotify-million-playlist-dataset-challenge)

## Data extraction
The first step will be to obtain keys to use. We'll need a [Spotify for developers](https://developer.spotify.com/) account for this. This is equivalent to a Spotify account and does not necessitate Spotify Premium. Go to the dashboard and select "create an app" from there. We now have access to the public and private keys required to use the API.

Now that we have an app, we can get a client ID and a client secret for this app. Both of these will be required to authenticate with the Spotify web API for our application, and can be thought of as a kind of username and password for the application. It is best practice not to share either of these, but especially don’t share the client secret key. To prevent this, we can keep it in a separate file, which, if you’re using Git for version control, should be Gitignored.

Spotify credentials should be stored the in the a `Spotify.yaml` file with the first line as the **credential id** and the second line as the **secret key**:
```python
Client_id : ************************
client_secret : ************************
```
To access this credentials, please use the following code:
```python
stream= open("Spotify/Spotify.yaml")
spotify_details = yaml.safe_load(stream)
auth_manager = SpotifyClientCredentials(client_id=spotify_details['Client_id'],
                                        client_secret=spotify_details['client_secret'])
sp = spotipy.client.Spotify(auth_manager=auth_manager)
```
## Code
### Reading1M_feature_extraction.ipynb
- This notebook reads the main.json files containing the playlists in order to train the model and generate recommendations.
- The loop_slices() function will go through as many slices as desired to extract the unique track URIs from the playlists for the content-based recommendation system.
- Using the Spotify API for Feature Extraction **(Audio Features, Track Release Date, Track Popularity, Artist Popularity, Artist Genres)** and Saving Results to a CSV Files and Errors to a Log File
```python
f = open('data/audio_features.csv','a')
e=0
for i in tqdm(range(0,len(t_uri),100)):
    try:
     track_feature = sp.audio_features(t_uri[i:i+100])
     track_df = pd.DataFrame(track_feature)
     csv_data = track_df.to_csv(header=False,index=False)
     f.write(csv_data)
    except Exception as error:
        e+=1
        r = open("audio_features_log.txt", "a")
        r.write(datetime.datetime.now().strftime("%d.%b %Y %H:%M:%S")+": "+str(error)+'\n')
        r.close()
        time.sleep(3)
        continue
r = open("audio_features_log.txt", "a")
r.write(datetime.datetime.now().strftime("%d.%b %Y %H:%M:%S")+" _________________________ "+"Total Number Of Errors : "+str(e)+" _________________________ "+'\n')
r.close()
f.close()
```
### Preprocessing.ipynb
- This notebook reads the extracted features and merges them into one dataframe.
- Handling missing extraction features and dropping duplicated and irrelevant columns
- Create five point buckets for track and artist popularity and 50 point buckets for the track release date.
```python
df['Track_release_date'] = df['Track_release_date'].apply(lambda x: int(x/50))
```
If I'm listening to music from the 1950s, I'd like the model to recommend music from the same era. 
### Modeling.ipynb
- Repeating the extraction features and preprocessing steps for the user's playlist (input)
- If a track from the user's playlist is missing from the dataset, it will be added automatically. 
- TfidfVectorizer was used for the Artist Genres (TF-IDF automatically assigns weights to metadata based on how frequently they appear). 

<img width="1017" alt="tfidf_4" src="https://user-images.githubusercontent.com/107134115/201203710-c1a48e8b-1365-4cc3-bba4-58a1102bafde.png">

- I was first using OneHotEncoder for **Track_release_date, Track_pop, Artist_pop** but I found no difference in the final result other than high memory usage.
- Converting a user playlist to a single vector 
- Cosine similarity is used to compare playlist vectors to song vectors to generate recommendations.

![image](https://user-images.githubusercontent.com/107134115/201203569-6bcd14fd-6704-4ad7-9577-44095bd65f74.png)

- I decided to go with three models. 

 **Model 1** which gives the genera more weight than the audio features

 **Model 2**  which gives equal weight to all features (as a result, playlist languages and genres are ignored)
 
 **Spotify Model**, which is available through the Spotify API

### Deployment
Please visit the following link to access the app's final version: https://huggingface.co/spaces/Longliveruby/Spotify-Recommendation-System
The website can be accessed and tested out there. Due to the limitations of file sizes and RAM limits, I decided to go with
[huggingface](https://huggingface.co/) because the free version is not severely limited.  

You can test the app on localhost by cloning the repository data, cd into the folder and run the following commands:
```python
cd Streamlit
streamlit run main.py
```
Installing dependencies:
```python
pip install -r requirements.txt
```

### Reference
- https://medium.com/analytics-vidhya/music-recommender-system-part-2-ff4c3f54cba3
- https://github.com/madhavthaker/spotify-recommendation-system
- https://spotipy.readthedocs.io/en/master/
