<template>
      <div>
        <div>
          <v-row align="center">
            <v-col cols="12" md="8">
           	<typewriter class="ma-3 d-flex align-center justify-center" style="min-height: 200px;"></typewriter>
            </v-col>
            <v-col cols="12" md="4" align="center">
              <v-card class="ma-3" hover
                >
                  <template v-slot:title >
                    <span class="leleo-card-title clock-font">{{formattedTime}}</span>
                  </template>
                  <template v-slot:subtitle>
                    <span style="font-weight: bold;">{{formattedDate}}</span>
                  </template>
                  <turntable :color1="configdata.color.turntablecolor1" :color2="configdata.color.turntablecolor2" />
              </v-card>
            </v-col>
          </v-row>
          
          
          
        </div>       
      </div>
</template> 

<script>
import typewriter from '../components/typewriter.vue';
import turntable from '../components/turntable.vue';
import { useDisplay } from 'vuetify'

export default {
    components: {
        typewriter,turntable
    },
    props: ['configdata','formattedTime','formattedDate'],
    setup() {
      const { xs,sm,md } = useDisplay();
      return {xs,sm,md};
    }
};
</script>

<style scoped>
@import url(/css/app.less);
@import url(/css/mobile.less);
</style>