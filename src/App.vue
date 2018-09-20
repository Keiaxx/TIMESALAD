<template>
    <v-app :dark="darktheme">

    <v-toolbar
      app
      :clipped-left="clipped"
    >

      <v-toolbar-title v-text="title"></v-toolbar-title>

    </v-toolbar>

    <v-content>
      <HelloWorld/>
    </v-content>

      <v-footer :fixed="fixed" app>
          &nbsp&nbsp&nbsp
          <i class="far fa-moon" v-if="!darktheme" @click="darktheme = true"></i>
          <i class="fas fa-moon" v-else @click="darktheme = false"></i>

          <v-spacer></v-spacer>

          <span>&copy; 2018 Adrian Gose &nbsp&nbsp&nbsp</span>
      </v-footer>
  </v-app>
</template>

<script>
import HelloWorld from './components/TimeTable'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data () {
    return {
        darktheme: false,
      clipped: false,
      drawer: true,
      fixed: false,
      items: [{
        icon: 'bubble_chart',
        title: 'Inspire'
      }],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Simple Time Clock'
    }
  }, beforeMount: function () {
        if (localStorage.getItem('darktheme') !== null) {
            var isDark = localStorage.getItem('darktheme');

            this.darktheme = (isDark === 'true');
        } else {
            localStorage.setItem('darktheme', this.darktheme);
        }
    },
    watch: {
        darktheme: function (after, before) {
            localStorage.setItem('darktheme', after);
        }
    }
}
</script>
