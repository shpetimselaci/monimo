<template>
    <header>
        <nav>
            <div style="cursor: pointer;" @click="$router.go(-1)" v-if="!isHome">
                <ArrowLeftIcon/>
            </div>
            <div class="logo-wrap">
                <a tabindex="0" @click="$router.push('/home')" class="logo">
                    <span style="font-size:80%; align-self: flex-end;">🦉</span>
                    <span>MONIMO 3</span>
                    <!-- <span class="sublogo">your anime best fren</span> -->
                </a>
            </div>
            <div class="search">
                <a tabindex="0" @click="showSearch()" :style="`opacity: ${!show_search ? 1 : 0}`">
                    <SearchIcon/>
                </a>
                <form v-show="show_search" @submit.prevent>
                    <input type="text" v-model="search" ref="searchInput" placeholder="Anime name" @blur="hideSearch()" @keydown.enter="searchAnimes()">
                    <button type="button" @click="searchAnimes()">
                        <SearchIcon/>
                    </button>
                </form>
            </div>
        </nav>
        <div class="progressbar" :style="progressBar" v-if="update_available">
        </div>
        <div class="updatebox" v-if="update_finished">
            <h4>Do you want to update?</h4>
            <button @click="updateApp()">Yes</button>
            <button @click="update_finished=!update_finished">Later</button>
          </div>
    </header>
</template>
<script>
import { ipcRenderer } from 'electron';
import {
    mapActions,
    mapMutations,
    mapState
} from 'vuex';
import { SearchIcon,ArrowLeftIcon } from 'vue-feather-icons'
let timeouts = 0;
let timeout;
export default {
    props: ['isHome'],
    components:{
        ArrowLeftIcon,
        SearchIcon
    },
    computed: {
        search: {
            get() {
                return this.$store.state.search_query;
            },
            set(value) {
                this.UPDATE_SEARCH_QUERY(value);
            },
        },
        isElectron() {
            return true; //navigator.userAgent.toLowerCase().indexOf('electron/') > -1;
        },
        percentage(){
          return -100 + this.downloaded_percentage;
        },
        progressBar(){
            return `transform: translateX(${this.percentage}%)`
        },
        ...mapState({
            window_mode: state => state.window_mode
        })
    },
    data: () => ({
        show_search: false,
        downloaded_percentage: 0,
        update_available: false,
        update_finished: false
    }),
    created(){
        ipcRenderer.on('update-available', (event, answer)=>{
          this.update_available = true;
        });
        ipcRenderer.on('error',()=>{
          this.update_available = false;
        });
        ipcRenderer.on('download-progress', (event, progressObj)=>{
          this.downloaded_percentage = parseFloat(progressObj.percent);
        });
        ipcRenderer.on('update-downloaded', (event, answer)=>{
          this.update_available = false;
          this.update_finished = true;
        });
    },
    methods: {
        showSearch() {
            if (timeouts) clearTimeout(timeout);
            this.show_search = true;
            setTimeout(() => {
                this.$refs.searchInput.focus();
            });
        },
        hideSearch() {
            timeout = setTimeout(() => {
                this.show_search = false
            }, 1000);
            timeouts++;
        },
        updateApp(){
          ipcRenderer.send('installUpdate');
        },
        ...mapActions(['getAnimes']),
        ...mapMutations(['UPDATE_SEARCH_QUERY', 'SET_WINDOW_MODE']),
        searchAnimes() {
            // this.getAnimes({
            //     order: 'relevance_desc',
            //     search: this.search
            // })
            if (this.search && this.search.length > 2 && this.$router.currentRoute.path !== `/results/${this.search}`) {
                this.$router.push({ path: `/results/${this.search}` });
                //console.log(this.$router.currentRoute);
            }
        },
    },
}
</script>
<style lang="scss">
.progressbar{
  position: absolute;
  height:5px;
  background-color: #2196f3;
  border-radius: 2px;
  width:100%;
  bottom:0;
  left:0;
  transition: 0.5s cubic-bezier(0.65, 0.05, 0.36, 1);
  &:hover:before{
    position: absolute;
    content: 'Updating app...';
    padding: 4px 2px;
    top: -26px;
    left: 10.8%;
    border-radius: 4px;
    font-size: 13px;
    margin-bottom: 20px;
    background-color: #2196f3;
    transition: 0.8s ease-in-out;
  }
}
.updatebox{
  h4{
    margin:0px;
  }
  position: absolute;
  bottom: 0;
  padding: 1px 10px 0px 10px;
  right: 0;
  background-color: #353535;
  text-align: center;
}
header {
  //-webkit-user-select: none;
  -webkit-app-region: drag;
  backdrop-filter: blur(21px);
  background-color: rgba(black, 0.75);
  padding: 20px;
  color: white;
  stroke: white;
  display: flex;
  position: fixed;
  top: 0;
  left: 0;
  height: 90px;
  width: 100%;
  z-index: 10;
  .search {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  form {
    display: flex;
    align-items: stretch;
    position: absolute;
    right: 0px;
    button {
      height: 100%;
      background: rgba(black, 0.8);
      svg {
        stroke: white;
      }
      border: none;
    }
  }
  input {
    background: rgba(black, 0.8);
    border: none;
    outline: dashed 1px rgba(white, 0.3);
    outline-offset: 2px;
    font-size: 1rem;
    padding: 5px;
    color: white;
  }
  nav {
    display: flex;
    align-items: center;
    flex: 1;
    > div.logo-wrap {
      flex: 1;
      display: inline-flex;
      justify-content: center;
      flex-shrink: 0;
    }
    .logo {
      text-align: center;
      font-size: 1.5rem;
      font-weight: bold;
      display: flex;
      align-items: center;
    }
    .sublogo {
      font-size: 1rem;
      line-height: 1;
      font-weight: 400;
    }
  }
}
header + * {
  padding-top: 90px;
}
</style>