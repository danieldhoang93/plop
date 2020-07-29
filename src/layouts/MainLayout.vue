<template>
  <q-layout view="hHh lpr fFf">
    <q-header elevated>
      <q-toolbar>
        

        <q-toolbar-title>
          Plop
          <div class="text-caption">Find the best place to do <span class="text-italic">your</span> business</div>
        </q-toolbar-title>

        
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      bordered
      content-class="bg-grey-1"
      :width="400"
      class="sideDrawer">
      
      <q-list>
        <q-form
        @submit="onSubmit"
        class="q-pa-sm">

        <q-input v-if="isError" filled v-model="errorMessage" dense disable />
        <q-input id="autocomplete" filled bottom-slots v-model="addressData" label="Enter address" dense>
          <template v-slot:append>
            <q-btn flat round color="primary" icon="place" @click="locate"/>
          </template>

        </q-input>

        <div>
          <q-btn label="Go" type="submit" color="primary"/>
        </div>
      </q-form>
      <q-separator />
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from 'components/EssentialLink.vue'

const linksData = [
  {
    title: 'Docs',
    caption: 'quasar.dev',
    icon: 'school',
    link: 'https://quasar.dev'
  }
];

export default {
  name: 'MainLayout',
  components: { EssentialLink },
  data () {
    return {
      leftDrawerOpen: false,
      essentialLinks: linksData
    }
  }
}
</script>

<style>
.sideDrawer {

}
</style>
