<template>
  <div class="fixed inset-0 w-full h-screen flex items-center justify-center bg-smoke-darker">
    <div v-click-outside.capture="onClose" class="w-8/12 bg-white shadow-lg rounded-lg px-8 pt-8 pb-4">
      <section class="flex">
        <div class="w-1/2 mr-4">
          <h1 class="font-semibold">FIRSTNAME*</h1>
          <input v-model="Firstname" type="text" placeholder="e.g. Artemis..." class="w-full border-b-4 p-2 mt-2 mb-4 focus:outline-none focus:border-app-primary">
          <h1 class="font-semibold">MIDDLENAME</h1>
          <input v-model="Middlename" type="text" placeholder="e.g. Marie..." class="w-full border-b-4 p-2 mt-2 mb-4 focus:outline-none focus:border-app-primary">
          <h1 class="font-semibold">LASTNAME</h1>
          <input v-model="Lastname" type="text" placeholder="e.g. Clairefond..." class="w-full border-b-4 p-2 mt-2 mb-4 focus:outline-none focus:border-app-primary">
          <h1 class="font-semibold">REGION</h1>
          <div class="inline-block relative w-64">
            <select v-model="SelectedRegion" class="w-full appearance-none border-b-4 p-2 mt-2 mb-4 focus:outline-none focus:border-app-primary">
              <option :value="null" disabled selected hidden>Please select a region...</option>
              <option v-for="region in Regions" :key="region.Id" :value="region">
                {{ region.Name }}
              </option>
            </select>
            <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pb-2 text-gray-700">
              <font-awesome-icon icon="angle-down" />
            </div>
          </div>
        </div>

        <div class="w-1/2">
          <label
            @mouseover="BannerIsHover = true"
            @mouseleave="BannerIsHover = false"
            class="flex flex-col items-center justify-center px-2 py-4 rounded-lg shadow-lg cursor-pointer opacity-100 hover:opacity-75"
          >
            <font-awesome-icon v-if="BannerIsHover" icon="camera" size="4x" class="absolute text-white opacity-50" />
            <img v-if="ImageUrl" :src="ImageUrl" alt="world banner" class="w-full h-full">
            <img v-else src="@/assets/-erwerwe-e.jpg" alt="world banner" class="w-full h-full rounded-lg">
            <input @change="onImageChange($event.target.files[0])" type="file" name="Banner" class="hidden">
            <small class="italic">Max resolution: 1080x1920</small>
          </label>
        </div>
      </section>

      <section class="pt-4 px-2">
        <button @click="onGoBack" class="float-left border-b-4 hover:border-app-primary py-2 px-4 font-normal">
          <ul class="flex">
            <li class="mr-2">
              <font-awesome-icon icon="arrow-left" />
            </li>
            <li>
              <p>Back</p>
            </li>
          </ul>
        </button>
        <button
          @click.prevent="onCreate"
          class="float-right shadow-md rounded py-2 px-4 bg-app-tertiary hover:bg-app-primary text-white font-semibold">
          Create
        </button>
      </section>
    </div>
  </div>
</template>

<script lang="ts">
/* tslint:disable:no-console */

import Vue from 'vue';
import Component from 'vue-class-component';

import moment from 'moment';
import { join } from 'path';
import * as editJsonFile from 'edit-json-file';
import * as vClickOutside from 'v-click-outside-x';

import Utils from '@/utils';
import { Character, AffiliatedRegion } from '@/api';

@Component({
  directives: {
    clickOutside: vClickOutside.directive,
  },
})
export default class CharacterNew extends Vue {
  private utils: Utils = new Utils();
  private Id: number;
  private Name: string = '';
  private ImageUrl: string = '';
  private Created: number = 0;

  private Firstname: string = '';
  private Middlename: string = '';
  private Lastname: string = '';

  private Regions: object[];
  private SelectedRegion: any;

  private BannerIsHover: boolean = false;

  constructor() {
    super();

    this.Id = this.$store.getters.getCharacterId;
    this.Regions = this.$store.getters.getRegionsName;
    this.SelectedRegion = null;
  }

  private created() {
    console.log(this.$route.name);
    console.log(this.$router.currentRoute);
  }

  private onCreate(): void {
    const filePath: string = join(
      this.utils.getCharactersSubfolderPath(),
      `${this.Id}.json`,
    );

    this.Created = moment().valueOf();
    this.Name = `${this.Firstname} ${this.Middlename} ${this.Lastname}`;

    const newCharacter: Character = new Character({
      Id: this.Id,
      ImageUrl: this.ImageUrl,
      Created: this.Created,
      Name: this.Name,
      Firstname: this.Firstname,
      Middlename: this.Middlename,
      Lastname: this.Lastname,
    });

    newCharacter.setAffiliatedRegion(this.SelectedRegion as AffiliatedRegion);

    this.utils.saveCharacter(filePath, newCharacter);

    const file = editJsonFile(this.utils.getConfigFilePath());

    this.$store.dispatch('incrementCharacterId');
    file.set('characterId', this.$store.getters.getRegionId);
    file.save();

    this.$store.dispatch('addCharacter', newCharacter)
    .then(() => {
      this.$router.replace({
        name: 'character-profile',
        params: {
          Id: this.Id.toString(),
        },
      });
    })
    .catch((err) => {
      throw err;
    });
  }

  private onImageChange(file: any) {
    const reader: FileReader = new FileReader();

    reader.onload = (event: any) => {
      this.ImageUrl = event.target.result;
    };
    reader.readAsDataURL(file);
  }

  private onGoBack() {
    if (window.history.length > 1) {
      this.$router.go(-1);
    }
  }

  private onClose() {
    if (window.history.length > 2) {
      this.$router.go(-2);
    }
  }
}
</script>

<style scoped>

</style>