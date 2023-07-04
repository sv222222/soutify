

<template>
   <div id="app">
    <header>
      <h1>SOUTIFY</h1>
    </header>
    <main>
      <section class="player">
        <h2 class="song-title">{{ current.title }} - <span>{{ current.artist }}</span></h2>
        <div class="controls">
          <input type="range" v-model="volume" min="0" max="1" step="0.01" @input="updateVolume">
          <button class="prev" @click="prev">Prev</button>
          <button class="play" v-if="!isPlaying" @click="play(current)">Play</button>
          <button class="pause" v-else @click="pause">Pause</button>
          <button class="next" @click="next">Next</button>
          <button class="shuffle" @click="shuffle">Shuffle</button>
          <button class="mute" @click="toggleMute">
    {{ isMuted ? 'Unmute' : 'Mute' }}
  </button>
        </div>
        <div class="repeat-controls">
          <button :class="{ active: repeatMode === 'none' }" @click="setRepeatMode('none')">No Repeat</button>
          <button :class="{ active: repeatMode === 'repeatOne' }" @click="setRepeatMode('repeatOne')">Repeat One</button>
          <button :class="{ active: repeatMode === 'repeatAll' }" @click="setRepeatMode('repeatAll')">Repeat All</button>
        </div>
        <div class="progress-bar">
          <progress v-bind:value="progressPercentage" max="100"></progress>
        </div>
        <div class="search">
  <input type="text" v-model="searchQuery" placeholder="Search for songs...">
  <button @click="search">Search</button>
</div>
      </section>
      <section class="playlist">
        <h3>SouDaGoat's Playlist</h3>
        <button v-for="song in filteredSongs" :key="song.src" @click="play(song)" :class="(song.src === current.src) ? 'song playing' : 'song'">
    {{ song.title }} by {{ song.artist }} ({{ song.duration }})
  </button>
      </section>
    </main>
  </div>
  <div v-if="loading" class="loading-indicator">Loading...</div>

</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      current: {},
      index: 0,
      abcd: 0,
      isPlaying: false,
      volume: 1,
      loading: false,
      searchQuery: '',
      repeatMode: 'none',
      currentTime: 0,
      isMuted: false,
       songs: [
        {
          title: 'Grateful',
          artist: 'Neffex',
          src: require('./assets/neffex-grateful.mp3'),
          duration: '4:33'
        },
        {
          title: 'Invincible',
          artist: 'Deaf Kev',
          src: require('./assets/deaf-kev-invincible.mp3'),
          duration: '3:02'
        },
        {
          title: 'IMY2',
          artist: 'Drake & Kid Cudi',
          src: require('./assets/Drake-Ft-Kid-Cudi-IMY2-(TrendyBeatz.com).mp3'),
          duration: '4:12'

        },
        {
          title: 'Fair Trade',
          artist: 'Drake & Travis Scott',
          src: require('./assets/Drake-Ft-Travis-Scott-Fair-Trade-(TrendyBeatz.com).mp3'),
          duration: '4:51'

        },
        {
          title: 'Knife Talk',
          artist: 'Drake, 21 Savage, & Project Pat',
          src: require('./assets/Drake-Knife-Talk-Ft-21-Savage-And-Project-Pat-(TrendyBeatz.com).mp3'),
          duration: '4:03'

        },
        {
          title: 'No Friends in the Industry',
          artist: 'Drake',
          src: require('./assets/Drake-No-Friends-In-The-Industry-1(TrendyBeatz.com).mp3'),
          duration: '3:26'

        },
        {
          title: 'Way 2 Sexy',
          artist: 'Drake, Future, & Young Thug',
          src: require('./assets/Drake-Way-2-Sexy-Ft-Future-And-Young-Thug-(TrendyBeatz.com).mp3'),
          duration: '4:18'

        },
        {
          title: 'Blinding Lights',
          artist: 'The Weeknd',
          src: require('./assets/The_Weeknd_-_Blinding_Lights_ghanavibez.com.mp3'),
          duration: '4:22'

        },
        {
          title: 'Save Your Tears',
          artist: 'The Weeknd',
          src: require('./assets/The_Weeknd_-_Save_Your_Tears_(Naijay.com).mp3'),
          duration: '4:08'

        }
        

        
        
      ],
      filteredSongs: [],
        
      player: new Audio()
     
    }
  },
  computed: {
    progressPercentage() {
      if (this.current.duration) {
        return (this.currentTime / parseFloat(this.current.duration)) * 100;
      } else {
        return 0;
      }
    },
  },
  methods: {
    play(song) {
      if (this.isPlaying) {
    // Song is already playing, no need to play again
    return;
  }
      if (typeof song.src != "undefined"){
        this.current = song;
        this.player.src = this.current.src;
        this.player.load();
        this.player.volume = this.volume;
      }
      
       
      while(this.abcd < this.songs.length)
      {
        if(this.current === this.songs[this.abcd])
        {
          this.index = this.abcd;
          this.abcd = 0;
          break;
        }
        this.abcd++;

      }
      this.loading = true; // Show loading indicator

      this.player.addEventListener('canplay', () => {
        this.loading = false; // Hide loading indicator once the audio can play
        if (this.isPlaying) {
      this.player.play();
    }
        });

      this.player.play().catch((error) => {
    console.error("Error occurred while playing the audio:", error);
    // Handle the error, e.g., display an error message to the user
        });
        this.player.addEventListener('timeupdate', () => {
        this.currentTime = this.player.currentTime;
      });
      this.player.addEventListener('ended', function(){
          this.index++;

          if(this.index>this.songs.length-1){
        this.index = 0;
          }
      this.current = this.songs[this.index];
      this.play(this.current);

      }.bind(this));
      this.isPlaying = true;
    },
    
    pause() {
      this.player.pause();
      this.isPlaying = false;
    },
    next() {
  switch (this.repeatMode) {
    case 'none':
      this.index++;
      if (this.index > this.songs.length - 1) {
        // End of playlist, stop playing
        this.pause();
        return;
      }
      break;
    case 'repeatOne':
      // Stay on the current song
      break;
    case 'repeatAll':
      this.index++;
      if (this.index > this.songs.length - 1) {
        // Reached the end of the playlist, go back to the first song
        this.index = 0;
      }
      break;
  }
  this.current = this.songs[this.index] || {};
  this.play(this.current);
},
    prev() {
        this.index--;
        if(this.index<0)
        {
          this.index = this.songs.length-1;
        }
        this.current = this.songs[this.index];
      this.play(this.current);
    },
    shuffle() {
  // Shuffles the songs array
  for (let i = this.songs.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [this.songs[i], this.songs[j]] = [this.songs[j], this.songs[i]];
  }

  // Sets the first song as the current song
  this.current = this.songs[0];
  this.index = 0;
  this.player.src = this.current.src;
  this.player.play();
},
search() {
    // Perform search logic based on the searchQuery value
    // Example: Filter the list of songs based on the search query
    this.filteredSongs = this.songs.filter(song => {
      // Adjust the condition based on your specific search logic
      return song.title.toLowerCase().includes(this.searchQuery.toLowerCase());
    });
  },
toggleMute() {
    this.isMuted = !this.isMuted;
    this.player.volume = this.isMuted ? 0 : this.volume;
  },
updateVolume() {
  this.player.volume = this.isMuted ? 0 : this.volume;
  },
  setRepeatMode(mode) {
  this.repeatMode = mode;
},

  },
  created () {
    this.current = this.songs[this.index];
    this.player.src = this.current.src;
   // this.player.play();
  }
  
}
</script>

<style>
/* Base styles */

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
}

header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 15px;
  background-color: #212121;
  color: #fff;
}

main {
  width: 100%;
  max-width: 768px;
  margin: 0 auto;
  padding: 25px;
}

.song-title {
  color: #53565a;
  font-size: 32px;
  font-weight: 700;
  text-transform: uppercase;
  text-align: center;
}

.song-title span {
  font-weight: 400;
  font-style: italic;
}

.controls {
  display: flex;
  justify-content: center;
  padding: 30px 15px;
  align-items: center;
}

.progress-bar {
  margin-top: 10px;
}

progress {
  width: 100%;
  height: 10px;
}

.loading-indicator {
  text-align: center;
  margin-top: 10px;
  font-size: 14px;
  color: #888;
}

button {
  appearance: none;
  background: none;
  border: none;
  outline: none;
  cursor: pointer;
}

button:hover {
  opacity: 0.8;
}

.buttons {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  margin-top: 15px;
}

.play,
.pause {
  font-size: 20px;
  font-weight: 700;
  padding: 15px 25px;
  margin: 5px;
  border-radius: 8px;
  color: #fff;
  background-color: #cc2e5d;
}

.next,
.prev {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 8px;
  color: #fff;
  background-color: #1db954;
}

.shuffle {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 8px;
  color: #fff;
  background-color: #4b7bec;
}

.mute {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 8px;
  background-color: #cc2e5d;
  color: #fff;
}

.song {
  display: block;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 8px;
  background-color: #f2f2f2;
  color: #53565a;
}

.song.playing {
  background-color: #1db954;
  color: #fff;
}
.repeat-controls button.active {
  background-color: #1db954;
  color: #fff;
}

/* Responsive styles */

@media (max-width: 768px) {
  .song-title {
    font-size: 24px;
  }

  .controls {
    flex-direction: column;
  }

  .buttons {
    flex-direction: row;
    justify-content: space-between;
  }

  .play,
  .pause,
  .next,
  .prev,
  .shuffle {
    font-size: 14px;
    padding: 10px 15px;
    margin: 5px;
  }

  
 
}


@media (max-width: 480px) {
  .song-title {
    font-size: 20px;
  }

  .controls {
    flex-direction: column;
  }

  .buttons {
    flex-direction: column;
    align-items: center;
  }

  .play,
  .pause,
  .next,
  .prev,
  .shuffle {
    font-size: 12px;
    padding: 8px 12px;
    margin: 5px;
  }

  .playlist {
    margin-top: 20px;
  }

  .song {
    font-size: 12px;
    padding: 8px;
  }
}
</style>
