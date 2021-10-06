# How-to-Build-a-Music-Playlist-with-React-Spotify-and-Fauna

*Written in connection with the Write with Fauna Program.*


Using React, Spotify API, and Fauna Database, we can build a personalized music playlist.

In this article, I will show the step-by-step procedures the average developer will follow to build this application. We will learn how to use Spotify Web API to authenticate users and search for music while using Fauna for data management.

# What is Spotify?

[Spotify](https://www.spotify.com) is a music streaming service provider. It provides a developer tool([Spotify Web API](https://api.spotify.com/)) that gives developers access to user and music-related data. In this article, we will be using Spotify for user Authentication and as a music catalog.

# Getting started with Spotify Web API

To use Spotify Web API in an application:  

- Create a Spotify account by signing up at [www.spotify.com](http://www.spotify.com/).
- Log in and go to the developer dashboard at https://developer.spotify.com/dashboard.
- Register your application by following the steps here: https://developer.spotify.com/documentation/general/guides/app-settings/#register-your-app.
- Take note of/save the `CLIENT ID` Spotify generated for the application.
- Make sure to set the redirect URI of the application to `http://localhost:3000/`. It would be best if you changed this when you are hosting the application in the public domain.

# What is Fauna?

Fauna is a cloud API that provides flexible, serverless, and friendly database instances. In this article, we will be using Fauna to store user and music-related data that we will use in the application.

# Getting started with Fauna DB

To use fauna DB:

- Create an account by signing up at: [https://dashboard.fauna.com/accounts/register](https://dashboard.fauna.com/accounts/register?utm_source=DevTo&utm_medium=referral&utm_campaign=WritewithFauna_MusicPlaylist_JAdewole)
![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630602238145_register.png)

## Creating a database for our application
- After registering, sign in to the dashboard and click `CREATE DASHBOARD`. 
- In the form that comes up, enter the database name and select the `Classic` region.
- Click the `CREATE` button.

## Creating collections

A collection is a group of related data stored in JSON objects.
For this application, we will need two collections: `users` and `playlists`.
To create these collections:

- Click on `NEW COLLECTION`.
- Enter the collection name.
- Click the `SAVE` button.

Repeat the steps above for the users and playlists collections.

## Creating Indexes

Indexes are references to documents other than the default references, used to enhance retrieval or finding of documents.
For this application, we will need two indexes: 

-  `playlist_for_user` to retrieve all playlists created by a particular user.
-  `user_by_user_id` to retrieve the document containing  a specific user’s data.

To create these indexes:

- Click on `NEW INDEX`.
- For the `playlist_for_user` index, enter the following details where applicable:
    1. Source Collection - playlist
    2. Index Name - playlist_for_user
    3. Terms - data.user_id
    4. Unique - `unchecked` 
    
- For the  `user_by_user_id`  index, enter the following details where applicable:
    1. Source Collection - users
    2. Index Name - user_by_user_id
    3. Terms - data.user_id
    4. Unique -  `checked` 

## Generating your Fauna Secret Key

This secret key is what connects our application to the database.
To generate your secret key:

- On the left-hand navigation menu, click on security.
- Click `NEW KEY`.
- Enter your key name.
- Click `SAVE` and a new key will be generated for you.

Make sure to save the secret somewhere safe.

# Building the application
## Setting up the application

To start with, I have created a starter application to bootstrap our building process.
You will need to clone it from this [GitHub repository](https://github.com/wolz-CODElife/Spotify-Playlist-Manager-With-FaunaDB) by running the following command in your terminal:

```bash
git clone https://github.com/wolz-CODElife/Spotify-Playlist-Manager-With-FaunaDB.git
```
In the folder downloaded, the following directories and files are present:

![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630787994385_directories.png)


The folders and files we will be working with are the ones inside `src` and they are:

1.  `app.js` : this file will contain the views(routes).
2.  `/utils/models.js` : this file is where we communicate with the Fauna database.
3.  `/utils/Spotify.js` : this file is where we communicate with Spotify web API.
4. The files inside `components` are react components that we use to build the user interface of the application.

## Installing the project’s dependencies

The application uses several node packages, which you will need to install to function well. To install these packages, run the following code in your terminal:

```bash
cd Spotify-Playlist-Manager-With-FaunaDB
npm install
```
## Installing FaunaDB node package

For our application to communicate with the database we created earlier, we will need to install the node package provided by fauna. To do this, open your terminal and type:

```bash
npm install faunadb
```
## Starting the application

For our application to run, open your terminal and type:

```bash
npm start
```
This should compile the react application and host it on `http://localhost:3000/`, the terminal should show this result:

![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630788605259_npm_start.png)


Now, open your browser and search `http://localhost:3000`, this should show the image below on your browser.

![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630788808486_fauna_home_screen.png)

## Creating our routes

Our application will have four routes: `Index`, `Create`, `MyCollections` and `Error`.

- Index: The home page that users will first see when they launch the application before authentication.
- Create: The page where users search for music and create a playlist of their desired music after authentication.
- MyCollections: The page where users browse and manage their saved playlists.
- Error: The page that comes up if the user goes to an undefined route.

We will define our routes by putting the following codes inside the  `App.js`.

```javascript
import React from 'react'
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Index from './components/Index';
import Create from './components/Create';
import Error from './components/Error'
import MyCollections from './components/MyCollections';
const App = () => {
    return (
    <Router>
        <Switch>
        <Route exact path="/" children={<Index />} />
        <Route path="/create" children={<Create />} /> 
        <Route path="/mycollections" children={<MyCollections /> } /> 
        <Route path="*" children={Error} />
        </Switch>
    </Router>
    )
}
export default App;
```

What we are doing here is checking the particular  `path`  a user is on then render the associated component as  `children`  props of  `Route` .

## Fetching Spotify data

We need three functions:  `getAccessToken` ,  `getUserId`  and  `search` .

-  `getAccessToken`  : this function sends a request to Spotify authorization API, if the user is accepts or authorizes Spotify to share his/her data with our application, Spotify will return a  `accesstoken`  which our application can later use to makes requests to the other Spotify API routes securely.
-  `getUserId`  : this function sends a request to Spotify, and if the  `accessToken`  is authenticated, Spotify returns the users data to our application.
-  `search`  : this function sends a requests with an argument  `term` , Spotify will return music tracks that fit the  `term`  of the user’s search.

```javascript
const clientId = "YOUR-SPOTIFY-CLIENT-ID"
const redirectUri = encodeURIComponent("http://localhost:3000/")
const scopes = encodeURIComponent("user-read-private user-read-email playlist-modify-public")
let accessToken
const Spotify = {
    getAccessToken : () => {
        if(localStorage.getItem('accessToken')){
            return JSON.parse(localStorage.getItem('accessToken'))
        }
        accessToken = window.location.hash
        .substring(1)
        .split('&')
        .reduce((initial, item) => {
            let parts = item.split('=')
            initial[parts[0]] = decodeURIComponent(parts[1])
            return initial
        }, {}).access_token
        if (accessToken) {            
            localStorage.setItem('accessToken', JSON.stringify(accessToken))
            return accessToken
        }
        else {
            const accessUrl = `https://accounts.spotify.com/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scopes}&response_type=token`
            window.location = accessUrl
        }
    },
    getUserId: () => {
        accessToken = Spotify.getAccessToken()
        if (accessToken) {
            const headers = { Authorization: `Bearer ${accessToken}` }
            return fetch("https://api.spotify.com/v1/me", { headers: headers })
            .then(response => response.json())
            .then(jsonResponse => {            
                if (jsonResponse) {
                    const { id, display_name, email, external_urls, images } = jsonResponse
                    const profile = {
                        user_id: id,
                        email: email,
                        name: display_name,
                        image: images[0].url,
                        url: external_urls.spotify
                    }
                    return profile
                }
            })
        }
    },
    search : (term) => {
        accessToken = Spotify.getAccessToken()
        if (accessToken) {
            const headers = {Authorization: `Bearer ${accessToken}`}
            return fetch(`https://api.spotify.com/v1/search?type=track&q=${term}`, {headers: headers})
            .then(response => { return response.json() })
            .then(jsonResponse => {
                if (!jsonResponse.tracks) {
                    return []
                }
                return jsonResponse.tracks.items.map(track => ({
                    id: track.id,
                    name: track.name,
                    artist: track.artists[0].name,
                    album: track.album.name,
                    image: track.album.images[1].url,
                    uri: track.uri
                }))
            })
        }
    }
}

export default Spotify
```
    
## Creating the models

Our application has six model functions:  `createUser` ,  `getUser` ,  `savePlaylist` ,  `getPlaylists` ,  `getPlaylist`  and  `deletePlaylist` .

-  `createUser`  : takes the user’s data and creates a new document in our Fauna database. If the there was a user registered with the same Spotify ID, the application will throw an error because, we set the  `user_by_user_id`  index to accept only unique  `user_id`  else, store the user data on the database and create a localstorage item which will contain the user’s details till the user logs out.
-  `getUser`  : accepts an argument  `user_id`  and queries the database using the  `user_by_user_id`  index.
-  `savePlaylist`  : takes the  `user_id` , and a list of music that the user has selected.
-  `getPlaylists`  ; takes the  `user_id`  and returns all the collections of playlists created by that user.
-  `getPlaylist`  : takes the  `id`  of a playlist and returns the list of music in that playlist.
-  `deletePlaylist`  : takes the  `id`  of a playlist and delete the collection.

To create our model,  `/utils/models.js`  will contain the following code:

```javascript
import faunadb, { query as q } from 'faunadb'
const client = new faunadb.Client({ secret: "YOUR-FAUNA-SECRET-KEY" })
export const createUser = async ({user_id, email, name, image, url}) => {
    try {
        const user = await client.query(
            q.Create(
                q.Collection('users'),
                {
                    data: {user_id, email, name, image, url}
                }
            )
        )
        localStorage.setItem('user', JSON.stringify(user.data))
        return user.data
    } catch (error) {
        return
    }
}
export const getUser = async (user_id) => {
    try {
        const user = await client.query(
            q.Get(
                q.Match(q.Index('user_by_user_id'), user_id)
            )
            )
        localStorage.setItem('user', JSON.stringify(user.data))
        return user.data
    }
    catch (error) {
        return
    }
}
export const savePlaylist = async (user_id, name, tracks) => {
    if(!name || !tracks.length){
        return 
    }
    try {
        const playlists = await client.query(
            q.Create(
                q.Collection('playlists'),
                {
                    data: {user_id, name, tracks}
                }
            )
        )
        return playlists.data
    } catch (error) {
        return
    }
}
export const getPlaylists = async (user_id) => {
    let playlistList = []
    try {
        const playlists = await client.query(
            q.Paginate(
                q.Match(q.Index('playlist_for_user'), user_id)
            )
        )
        for (let playlistID of playlists.data) {
            let playlist = await getPlaylist(playlistID.value.id)
            playlistList.push(playlist)
        }
        return playlistList
    } catch (error) {
        return
    }
}
export const getPlaylist = async (id) => {
    try {
        
        const playlist = await client.query(
            q.Get(q.Ref(q.Collection('playlists'), id))
        )
        playlist.data.id = playlist.ref.value.id
        return playlist.data
    } catch (error) {
        return
    }
}
    
export const deletePlaylist = async (id) => {
    try {   
        const playlist = await client.query(
            q.Delete(
                q.Ref(q.Collection('playlists'), id)
            )
        )
        playlist.data.id = playlist.ref.value.id
        return playlist.data
    } catch (error) {
        return
    }
}
```
## Creating the Index Page

When the application is initially run, or a user goes to the  `/` route, we expect the application to render an authentication page.

When the Index component loads: if the user has logged in, we redirect the user to ‘/create’ using the  `useHistory`   hook else, we want to display the content of the Index component. 

The login and signup buttons have an onClick event listener, which calls their appropriate functions when clicked.

The  `Signup`  function gets the user’s Spotify ID from  `Spotify.getUserId`  function, then tries to create a new user on our Fauna database with the Spotify ID that was gotten. If the ID has been registered before an error message is displayed else, we redirect the user to ‘/create’ route.

The  `Login`  function also gets the user’s Spotify ID from  `Spotify.getUserId`  function, then query the Fauna database for a user with that ID. If the ID is not found as a user, display an error message else, redirect to ‘/create’ route.

 `/components/Index.js`  will contain the following code:
 
```javascript
import React from 'react'
import { useHistory } from 'react-router-dom'
import { createUser, getUser } from '../utils/model'
import Spotify from '../utils/Spotify'
const Index = () => {
    const history = useHistory()
    
    if (localStorage.getItem('user')) {            
        history.push('/create')
    }
    const Signup = () => {
        Spotify.getUserId().then((newUserData) => {
            createUser(newUserData)
            .then(req => {
                if (req)
                    history.push('/create')
                else
                    alert("Spotify account already registered!")
            })
            .catch((err) => console.log(err.message))
        })
    }
    
    const Login = () => {
        Spotify.getUserId().then((newUserData) => {
            getUser(newUserData.user_id)
            .then(req => {
                if (req)
                    history.push('/create')
                else
                    alert('Spotify account not found! Signup first')
            })
            .catch((err) => console.log(err.message))
        })
    }
    return (
        <>
            <div className="container">
                <br /><br /><br />
                <h1>MusicBuddy</h1>
                <br /><br />
                <span className="btn" onClick={() => Login()}>Login</span>
                <br /><br />
                <p>OR</p>
                <span className="btn" onClick={() => Signup()}>SignUp</span>
            </div>
        </>
    )
}
export default Index
```
    
![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630901729746_index.png)

## Creating the NavBar component

The NavBar component is where we will have user profile, navigation links and a logout button.

The NavBar accepts a props called  `userData` . We also have a state used to check if the user’s profile dropdown is visible or not. The div with atrribute  `className="dropDown"`  has a onMouseEnter and onMouseLeave which changes the  `userProfile`  state to true or false. When  `userProfile`  is true, the `<ul>` tag containing the user’s profile is rendered else, it is hidden.

The logout button has an onClick event listener, which clears the localstorage.

`components/NavBar.js`  will contain the following code:
 
```javascript
import React, { useState} from 'react'
import { Link } from 'react-router-dom'
import userImg from '../assets/justin.PNG'
const NavBar = ({ userData }) => {
    const [userProfile, setUserProfile] = useState(false)
    return (
        <>
            <div >
                <div className="dropDown" onMouseEnter={() => setUserProfile(!userProfile)} onMouseLeave={() => setUserProfile(false)}>
                    <img src={userData?.image || userImg} alt="user" />
                    {userProfile && <ul>
                        <li><h3>{ userData?.name || 'John Doe' }</h3></li>
                        <li>
                            <p >
                                <a href={userData?.url || '/'} target="_blank" rel="noopener noreferrer">{'Profile >>'}</a>
                            </p>
                        </li>
                    </ul>}
                </div>
                <div>
                    <Link to="/" className="btn">Home</Link>
                    <Link to="/mycollections" className="btn">Collections</Link>
                    <Link to="/" className="btn" onClick={() => localStorage.clear()}>Logout</Link>
                </div>
            </div>
        </>
    )
}
export default NavBar
```
    
![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630901750095_navbar.png)

## Creating the create new playlist page

This component contains three other components:  `NavBar` ,  `PlayList`  and  `SearchResults` .

-  `SearchResults`  allows users to search for music in our application and fetch a result from Spotify API.
-  `PlayList`  allows users create a playlist of some selected music and store them in the database.

 `/components/create.js`  will contain the following code:
 
```javascript
import React, { useState, useEffect } from 'react'
import PlayList from './PlayList'
import SearchResults from './SearchResults'
import Spotify from '../utils/Spotify'
import NavBar from './NavBar'
import { useHistory } from 'react-router-dom'
import { savePlaylist } from '../utils/model'
const Create = () => {
    const history = useHistory()
    const [userData, setUserData] = useState(JSON.parse(localStorage.getItem("user")))
    useEffect(() => {
        if (!localStorage.getItem('user')) {
            history.push('/')       
        }
        setUserData(JSON.parse(localStorage.getItem("user")))
    }, [history])
    const [searchResults, setSearchResults] = useState([])
    const [playListName, setPlayListName] = useState("")
    const [playListTracks, setPlayListTracks] = useState([])
    const search = (term) => {
        if (term !== "") {
            Spotify.search(term).then((searchResults) => setSearchResults(searchResults))
        }
        else {
        document.querySelector("#searchBar").focus()
        }
    }
    const addTrack = (track) => {
        if (playListTracks.find((savedTrack) => savedTrack.id === track.id)) {
        return
        }
        const newPlayListTracks = [...playListTracks, track]
        setPlayListTracks(newPlayListTracks)
    }
    const removeTrack = (track) => {
        const newPlayListTracks = playListTracks.filter((currentTrack) => currentTrack.id !== track.id)
        searchResults.unshift(track)
        setPlayListTracks(newPlayListTracks)
    }
    const removeTrackSearch = (track) => {
        const newSearchResults = searchResults.filter((currentTrack) => currentTrack.id !== track.id)
        setSearchResults(newSearchResults)
    }
    const doThese = (track) => {
        addTrack(track)
        removeTrackSearch(track)
    }
    const updatePlayListname = (name) => {
        setPlayListName(name)
    }
    const savePlayList = (e) => {
        e.preventDefault()
        if (playListName !== "") {
            alert('Playlist added successfully...')
            savePlaylist(userData.user_id, playListName, playListTracks)
            .then(req => {
                if (req) {
                    setPlayListName("")
                    setPlayListTracks([])
                }
            })
        }
        else {
        document.querySelector('#playListName').focus()
        }
    }
    return (
        <>
            <NavBar userData={userData}/>
            <div className="container">
                <h1 >MusicBuddy</h1>
                <article className="section">
                    <SearchResults search={search} searchResults={searchResults} onAdd={doThese} />
                    <PlayList playListTracks={playListTracks} playListName={playListName} onNameChange={updatePlayListname} onRemove={removeTrack} onSave={savePlayList} />
                </article>
            </div>
        </>
    )
}
export default Create
```


## Creating the search results component

This component contains a  `SearchBar`  and  `TrackList`  component.

-  `SearchBar`  component contains a form for users to search for random songs from Spotify.
-  `TrackList`  component displays the search results.

 `/components/SearchResults.js`  should contain the following code:
 
```javascript
import React, { useState } from 'react'
import TrackList from './TrackList'
const SearchResults = ({ search, searchResults, onAdd }) => {
    return (
        <>
            <div className="trackList">
                <SearchBar onSearch={search} />
                <TrackList tracks={searchResults} onAdd={onAdd} />
            </div>
        </>
    )
}
const SearchBar = ({ onSearch }) => {
    const [term, setTerm] = useState("");
    const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(term);
    };
    return (
    <>
        <form className="form" onSubmit={handleSubmit}>
        <input
            id="searchBar"
            type="text"
            placeholder="Song, album or artist name"
            onChange={(e) => setTerm(e.target.value)}
        />
        <button className="btn" onClick={handleSubmit}>
            SEARCH
        </button>
        </form>
    </>
    );
};
    
export default SearchResults
```

## Creating the playlist components

This component contains a form and  `TrackList`  component.

- The form is used to set a name for the playlist the user is creating.
-  `TrackList`  displays a list of music to be included in the playlist, that the user will create.

 `/components/PlayList.js`  will contain the following code:
 
```javascript
import React from "react";
import TrackList from "./TrackList";
const PlayList = ({ onNameChange, playListTracks, playListName, onRemove, onSave }) => {
    return (
    <>
        <div className="trackList">
        <form className="form" onSubmit={onSave}>
            <input id="playListName" type="text" onChange={(e) => onNameChange(e.target.value)} defaultValue={playListName} placeholder="Playlist Name" />
            {(playListTracks.length > 0) &&        
            <button className="btn" onClick={onSave}>
                Save to Collections
            </button>}
        </form>
        <TrackList tracks={playListTracks} isRemoval={true} onRemove={onRemove} />
        </div>
    </>
    );
};
export default PlayList;
```
    
![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630965252781_playlist.png)


So far, you should have observed that the  `SearchResults`  and the  `PlayList`  components imported  `TrackList`  .

## Creating track list component

This component contains  `Track`  component which is mapped to each item of the list of tracks.
 `/components/TrackList.js`  will contain the following code:
 
```javascript
import React from 'react'
import Track from './Track'
import Img from '../assets/omo.png'
const TrackList = ({ tracks, onAdd, isRemoval, onRemove }) => {
    return (
        <>
            {(tracks.length > 0) &&
                <div className="playList">
                    {tracks.map((track) => {
                        return (
                            <Track key={track.id} track={track} onAdd={onAdd} isRemoval={isRemoval} onRemove={onRemove} />
                        )
                    })}
                </div >
            }
            {(tracks.length === 0) &&
                <div className="playList">
                <img src={Img} alt="Oops!" />
                    <h3>Oops! No Tracks founds</h3>
                    <p>Search and add for a track</p>
                </div>
            }
        </>
    )
}
export default TrackList
```
## Creating the track component
This component accepts a track’s data as an object and renders a Spotify player in `<iframe>`. It also contains a  TrackAction that allows users to add or remove a track from the tracklist.
 `/components/Track.js`  will contain the following code:
```javascript
import React, { useState, useEffect } from 'react'
import bgImg from '../assets/justin.PNG'
const Track = ({ track, onAdd, onRemove, isRemoval }) => {
    const [trackBg, setTrackBg] = useState('')
    useEffect(() => {
        track.image? setTrackBg(track.image) : setTrackBg(bgImg)
    }, [track.image])
    const addTrack = () => onAdd(track)
    const removeTrack = () => onRemove(track)
    return (
        <ul className="track">
            <li>
                <div>
                    <div className="item" >                        
                        <div>
                            <h3>{track.name}</h3>
                            {track.artist} | {track.album}
                        </div>
                        {
                            onAdd || onRemove ?
                                <TrackAction isRemoval={isRemoval} removeTrack={removeTrack} addTrack={addTrack} />
                            :
                                ""
                        }
                    </div>
                </div>
            </li>
            <li>
                <iframe src={"https://open.spotify.com/embed/track/" + track.id} width="100%" height="80" frameBorder="0" allowtransparency="True" allow="encrypted-media" title="preview" />
            </li>
        </ul>
    )
}
const TrackAction = ({ isRemoval, removeTrack, addTrack }) => {
    return (
        <>
            {
                isRemoval ?
                    <button className="btn" onClick={removeTrack}> - </button>
                :
                    <button className="btn" onClick={addTrack}> + </button>
            }
        </>
    )
}

export default Track
```
## Creating the user’s playlist collection page

This component contains a list of all the playlists a user has saved to the Fauna database.

The  `getPlaylists`  function gets all the playlists that the authenticated user creates. 

The playlists tracks are hidden by default until the user clicks on a particular playlist, then the  `togglePlaylist`  function sets the clicked playlist to active, then the tracks that belong to the active playlist are rendered. 

The  `removePlaylist`  function takes a playlist’s id and deletes it from the database.
 `/components/MyCollections.js`  will contain the following code:
  
 ```javascript
import React, { useState, useEffect } from "react";
import NavBar from "./NavBar";
import { useHistory } from "react-router-dom";
import { deletePlaylist, getPlaylists } from "../utils/model";
import bgImg from '../assets/justin.PNG'
import Track from './Track'
const MyCollections = () => {
    const history = useHistory();
    const [userData, setUserData] = useState(JSON.parse(localStorage.getItem("user")));
    const [playlists, setPlaylists] = useState([])
    const [activePlaylist, setactivePlaylist] = useState()
    useEffect(() => {
    if (!localStorage.getItem("user")) {
        history.push("/");
    }
    getPlaylists(userData?.user_id)
    .then(req => {
        return setPlaylists(req)
    })
    .catch((err) => console.log(err.message))
    if (!userData) {
        setUserData(JSON.parse(localStorage.getItem("user")))
    }
    }, [userData, history]);
    
    const togglePlaylist = (id) => {
        if (activePlaylist === id) {
            setactivePlaylist()
        }
        else {
            setactivePlaylist(id)
        }
    }
    const removePlaylist = (playlist) => {
        deletePlaylist(playlist.id)
        .then(req => {
            const newPlaylist = playlists.filter((list) => list.id !== playlist.id)
            playlists.unshift(playlist)
            return setPlaylists(newPlaylist)
        })
        .catch((err) => console.log(err.message))
    } 
    return (
    <>
        <NavBar userData={userData} />
        <div className="container">
        <h1 >
            My Collections
        </h1>
        <article className="section">            
            <div className="trackList">
                <div className="playList">
                    {playlists.length ?
                        playlists?.map((playlist) => { return (
                            <ul className="track" key={playlist.id}>
                                <li onClick={() => togglePlaylist(playlist.id)}>
                                    <div >
                                        <div className="item" >                        
                                            <div>
                                                <h3>{playlist.name}</h3>
                                            </div>
                                            <button className="btn" onClick={(e) => {
                                                e.preventDefault()
                                                removePlaylist(playlist)
                                            }}> Delete </button>
                                        </div>
                                    </div>
                                </li>
                                {activePlaylist === playlist.id &&
                                    <div >
                                        {playlist.tracks.map((track) => {
                                            return (
                                                <Track
                                                    key={track.id}
                                                    track={track}
                                                />
                                        )})}
                                    </div>
                                }
                            </ul>
                        )
                        })
                    :
                        <h2 >No Playlist saved . . .</h2>
                    }
                </div>
            </div>
        </article>
        </div>
    </>
    );
};
export default MyCollections;
```
    
![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630965286608_collection.png)


With the components set up this way, our application should run correctly.
We won’t stop there. Notice: there is an error if we go to a route that we did not define. That is because we have to create an error-handling component.

## Error handling

We will create a component that will be rendered when user goes to any route we did not define.
 `/components/Error.js`  will contain the following code:

```javascript
import React from 'react'
import { Link } from 'react-router-dom' 
const Error = () => {
    return (
        <div >
            <h1> Oops! Page Not found. </h1>
            <h3><Link to="/create">Go back to safety</Link></h3>
        </div>
    )
}
export default Error
```  
![](https://paper-attachments.dropbox.com/s_F833050C170A103F41D948F4B16A6398DAC29064C0E2580E28394213DDB28D98_1630965336984_error.png)

# Conclusion

Having created this application successfully integrating Fauna and Spotify in React, we have learned how to authenticate users without emails and passwords by using the Spotify Web API and storing user’s data using Fauna database. We also explored the Spotify Web API search endpoint and how to handle requests and responses from the API while using the Fauna database as a storage medium.

You can download the source code of the working application from my [GitHub repository](https://github.com/wolz-CODElife/Spotify-Playlist-Manager) or visit a [demo](https://spotify-wc-playlist-manager.web.app/) here. You can also contact me via [Twitter](https://twitter.com/wolz_codelife). 

*Written in connection with the Write with Fauna Program.* 

