<template>
  <v-app class="vapp-fullscreen-background" :class="{ 'radius-before': !xs }"
  :style="xs?{height: '100%',width: '100%',top: '0',left:'0'}:(sm?{height: '98%',width: '98%',top: '1%',left:' 1%'}:{height: '96.6%',width: '99%',top: '1.7%',left:' 0.5%'})">
    <transition name="fade">
      <div class="loading" v-show="isloading">
        <loader></loader>
      </div>
    </transition>

    <video autoplay loop muted class="video-bg" id="bg-video" ref="VdPlayer"
    :style="xs?{height: '100%',width: '100%',top: '0',left:'0'}:(sm?{height: '98%',width: '98%',top: '1%',left:' 1%','border-radius': '16px'}:{height: '96.6%',width: '99%',top: '1.7%',left:' 0.5%','border-radius': '16px',})">
        <source :src=videosrc type="video/mp4">
    </video>

    <div class="floating-switch-container">
      <v-switch
        v-model="isClearScreen"
        inset
        :style="xs?{'transform':'scale(0.6) translateX(15%)'}:{}"
        class="floating-switch"
        @mouseover="expandSwitch"
        @mouseleave="collapseSwitch"
      ></v-switch>
    </div>
    
    <div v-show="!isloading && !isClearScreen" :style="xs||sm?{'overflow-y': 'auto','overflow-x': 'hidden'}:{}">
        <v-row>
            <v-col cols="12" md="4" lg="3" class="leleo-left" align="center">
              <v-avatar class="leleo-left-avatar" :size="xs||sm?120:140" :style="xs||sm?{'margin-top': '0'}:{'margin-top': '2rem'}" @mouseenter="musicplayershow(1)" @mouseleave="musicplayershow(0)">
                  <v-img :class="{'leleo-spin':isPlaying}"
                  alt="Leleo"
                  :src=configdata.avatar
                  ></v-img>
                  <!-- 由于当ismusicplayer显示后，fadein无效果，所以需要设置一个过渡动画 -->
                  <transition name="fade">
                  <v-card v-show="ismusicplayer" class="musicplayer" :class="{'fade-in':ismusicplayer}" variant="tonal">
                      <div v-if="audioLoading" class="loading-spinner">
                          <v-progress-circular indeterminate></v-progress-circular>
                      </div>
                      <span ref="audiotitle" class="musicplayer-text"
                        style="top: 1.6rem;font-weight: bolder;"
                      >{{ musicinfo?.[0]?.title }}</span>
                      <span ref="audioauthor" class="musicplayer-text"
                        style="bottom: 1.4rem;"
                      >{{ musicinfo?.[0]?.author }}</span>
                      <audio v-show="false" ref="audioPlayer" :src="musicinfo?.[0]?.url"
                      @waiting="onWaiting"
                      @canplay="onCanPlay">
                      </audio>
                      <v-btn :size="xs||sm?22:30" color="#999999" icon @click="previousTrack()">
                      <v-icon>mdi-skip-previous</v-icon>
                      </v-btn>
                      <v-btn :size="xs||sm?35:48" color="#999999" icon @click="togglePlay()">
                      <v-icon>{{ isPlaying? 'mdi-pause' : 'mdi-play' }}</v-icon>
                      </v-btn>
                      <v-btn :size="xs||sm?22:30" color="#999999" icon @click="nextTrack()">
                      <v-icon>mdi-skip-next</v-icon>
                      </v-btn>
                  </v-card>
                  </transition>
                </v-avatar>

                <v-card class="ma-5 pa-2 leleo-left-card" variant="tonal" :max-width="xs?270:300" style="text-align: center;">
                    <template v-slot:title>
                    <span>Tags</span>
                    </template>
                    <v-chip v-for="item in personalizedtags" density="compact" link class="ma-1" size="small">
                    {{item}}
                    </v-chip>
                </v-card>

                

                <v-container class="leleo-left-socialIconsContainer">
                    <v-row align="center" justify="center">
                    <v-col class="pa-1" cols="auto" v-for="item in socialPlatformIcons">
                        <v-btn :size="xs?25:33" variant="tonal" color="var(--leleo-vcard-color)"
                        class="ma-1 leleo-social-bticon"
                        icon
                        :href="item.link" target="_blank"
                        >
                    <v-icon :icon=item.icon :size="xs?20:25" class="social-bticon-icon"></v-icon></v-btn>
                    </v-col>
                    
                    <v-col class="pa-1" cols="auto">
                        <v-btn :size="xs?25:33" variant="tonal" color="var(--leleo-vcard-color)"
                        class="ma-1 leleo-social-bticon"
                        icon
                        @click="dialog1 = true; tab = 'tab-2'"
                        >
                            <v-icon :size="xs?20:25" icon="mdi-cog"></v-icon>
                        </v-btn>
                    </v-col>
                    </v-row>
                </v-container>
            </v-col>

            <v-col cols="12" md="8" lg="9" style="height: 100vh;" :style="xs||sm ?{}:{'overflow': 'auto'}">
                <homeright :configdata=configdata :formattedTime=formattedTime 
                :formattedDate=formattedDate></homeright>
            </v-col>
        </v-row>
    </div>

    <v-dialog
        v-model="dialog1"
        width="1000"
        heihght="700"
      >
      <v-card elevation="3" style="backdrop-filter: blur(10px);">
        <v-tabs
          v-model="tab"
          :items="tabs"
          align-tabs="center"
          height="60"
          slider-color=var(--leleo-vcard-color)
        >
          <template v-slot:tab="{ item }">
            <v-tab
              :prepend-icon="item.icon"
              :text="item.text"
              :value="item.value"
              class="text-none"
            ></v-tab>
          </template>
          
          <template v-slot:item="{ item }">
            <v-tabs-window-item :value="item.value" class="pa-4">
              <div v-if="item.value=='tab-3' && musicinfoLoading" class="loading-spinner" align="center">
                  <v-progress-circular indeterminate></v-progress-circular>
              </div>
              <!-- 通过组件绑定不同tab项的组件 -->
              <component v-if="item.value!='tab-3' || (item.value=='tab-3' && !musicinfoLoading)" :is=item.component @cancel="handleCancel" 
              :musicinfo="item.value=='tab-3'?musicinfo:[]"
              :currentIndex="item.value=='tab-3'?playlistIndex:null"
              :isPlaying="item.value=='tab-3'?isPlaying:null"
              :audioPlayer="item.value=='tab-3'?audioPlayer:null"
              :fromLyrics="item.value=='tab-3'?lyrics:null"
              :audioLoading="item.value=='tab-3'?audioLoading:null"
              @update:current-index="updateCurrentIndex"
              @update:is-playing="updateIsPlaying"
              @update:current-lyrics="updateLyrics"
              ></component>
            </v-tabs-window-item>
          </template>
        </v-tabs>
      </v-card>
      </v-dialog>

      
  </v-app>
</template>

<script src="./app.js"></script>
<style scoped>
  @import url(/css/app.less);
  @import url(/css/mobile.less);
</style>