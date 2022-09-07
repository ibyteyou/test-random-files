<script>
  import './assets/fonts/fontawesome/css/all.css'
  import Hls from 'hls.js'
  import fetch from './vendor/fetchInterceptor'

  const Hostname = `${window.location.protocol}//${window.location.hostname}`

  export default {
    name: 'app',
    data: () => ({
      ready: false,
      artist: null,
      title: null,
      time: null
    }),
    methods: {
      nextTrack () {
        fetch(`${Hostname}:9099/next-track`)
      },
      prevTrack () {
        fetch(`${Hostname}:9099/prev-track`)
      },
      async updateTrackInfo () {
        const { artist, time, title } = await fetch(`${Hostname}:9099/track-info`).then(res => res.json()).catch(err => console.error(err))
        this.artist = artist
        this.time = time
        this.title = title
        this.ready = true
      }
    },
    async mounted () {
      this.updateTrackInfo()
      setInterval(() => {
        this.updateTrackInfo()
      }, 1000)


      const iOS = ['iPad Simulator', 'iPhone Simulator', 'iPod Simulator', 'iPad', 'iPhone', 'iPod'].includes(navigator.platform) || (navigator.userAgent.includes("Mac") && "ontouchend" in document)
      function playIconSetPause () {
        const playIcon = document.getElementById('playIcon');
        playIcon.classList.remove('fa-circle-play');
        playIcon.classList.add('fa-circle-pause');
      }
      function playIconSetPlay () {
        const playIcon = document.getElementById('playIcon');
        playIcon.classList.remove('fa-circle-pause');
        playIcon.classList.add('fa-circle-play');
      }
      if (Hls.isSupported()) {
        const video = document.getElementById('video');
        const playButton = document.getElementById('playButton');
        const hls = new Hls();
        // bind them together
        hls.attachMedia(video);
        hls.on(Hls.Events.MEDIA_ATTACHED, function () {
          console.log('video and hls.js are now bound together !');
          hls.loadSource('live/musify/index.m3u8');
          hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
            console.log(
              'manifest loaded, found ' + data.levels.length + ' quality level'
            );
            if (!iOS) {
              playButton.addEventListener('click', () => {
                video.play().then(() => {
                  console.log('play video stream');
                  playIconSetPause();
                  playButton.addEventListener('click', event => {
                    video.pause()
                    playIconSetPlay()
                    event.preventDefault()
                    event.stopPropagation()
                  }, { once: true, capture: true })
                }).catch(e => {
                  console.error(e)
                })
              }, false)
            }
          });
        });
      } else {
        // var audioIos = document.getElementById('audio-ios');
        const videoIos = document.getElementById('video-ios');
        const playButton = document.getElementById('playButton');
        playButton.addEventListener('click', () => {
          videoIos.play().then(() => {
            console.log('success audio stream');
            playIconSetPause();
            playButton.addEventListener('click', event => {
              videoIos.pause()
              playIconSetPlay()
              event.preventDefault()
              event.stopPropagation()
            }, { once: true, capture: true })
          }).catch(e => {
            console.error(e)
          });
        }, false)
        console.log('hls.js not supported, flv in development =/')
      }
    }
  }
</script>
