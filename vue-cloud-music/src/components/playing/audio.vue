<template>
  <div>
    <audio class="audio" ref="audio" :src="address" autoplay="autoplay" @play="onPlay" @timeupdate="timeUpdate" @ended="end"></audio>
  </div>
</template>

<script>
  import {getMusicUrl} from '@/common/js/axiosType/getAxiosType.js';
  import {UPDATE_CURRENTTIME, UPDATE_DURATION, PLAY_IRREGULAR, PLAY_NEXT} from '@/store/mutationType.js'
  import {mapMutations} from 'vuex'
  export default {
    name: 'audios',
    data () {
      return {
        address: '',
        currentTime: '',
        duration: '',
      }
    },
    computed: {
      id () {
        return this.$store.state.playingId
      },
      isPlaying () {
        return this.$store.state.isPlaying
      },
      jump () {
        return this.$store.state.jump
      },
      playMode () {
        return this.$store.state.playMode
      }
    },
    methods: {
      timeUpdate (e) {
//        console.log(e);
        let min = parseInt((e.path[0].currentTime / 60), 10);
        let sec = (e.path[0].currentTime % 60).toFixed(0);
        if (sec < 10) sec = `0${sec}`;
        this.currentTime = `${min}:${sec}`;
        this.UPDATE_CURRENTTIME({current: this.currentTime, unFixedTime: e.path[0].currentTime});
      },
      onPlay () {
        let min = (this.$refs.audio.duration / 60).toFixed(0);
        let sec = (this.$refs.audio.duration % 60).toFixed(0);
        if (sec < 10) sec = `0${sec}`;
        this.duration = `${min}:${sec}`;
        this.UPDATE_DURATION({duration: this.duration, unFixedDuration: this.$refs.audio.duration})
      },
      end () {
        switch (this.playMode) {
          //顺序播放，直接向store提交下一首
          case 0:
            this.PLAY_NEXT();
            this.pauseMusic = false;
            break;
          //单曲循环，这里偷懒直接把进度条拉到最开始的地方
          case 1:
            this.$refs.audio.currentTime = 0;
            this.$refs.audio.play();
            break;
          //随机播放，向store提交请求
          case 2:
            this.PLAY_IRREGULAR();
            this.pauseMusic = false;
            break;
        }
      },
      ...mapMutations([
        'UPDATE_CURRENTTIME',
        'UPDATE_DURATION',
        'PLAY_NEXT',
        'PLAY_IRREGULAR'
      ])
    },
    watch: {
      id () {
        getMusicUrl(this.id).then(result => {
          console.log(result);
          this.address = result.data.data[0].url;
        });
      },
      isPlaying () {
        if (this.isPlaying === true) {
          this.$refs.audio.play();
          return
        }
        this.$refs.audio.pause();
      },
      jump () {
        this.$refs.audio.currentTime = this.jump
      }
    }
  }
</script>

<style lang="less" scoped>
  .audio {
    display: none;
  }
</style>
