<template>
  <div class="">
    <profile-back-button @click.native="onGoBack" class="" />
    <div class="w-full max-h-screen pb-20 force-overflow-main">
      <section class="w-full relative">
        <label
          @click="onEditImageUrl"
          @mouseover="BannerIsHover = true"
          @mouseleave="BannerIsHover = false"
          class="w-full flex flex-col items-center justify-center bg-app-primary shadow-lg rounded-lg cursor-pointer opacity-100 hover:opacity-75"
        >
          <img
            v-if="imageUrl"
            :src="ImageUrl"
            alt="world banner"
            class="flex w-full relative bg-no-repeat bg-cover m-0 p-0 items-center justify-around rounded">
          <img
            src="@/assets/jan-urschel-gis-ju-deepdive01-d1.jpg"
            alt="default world banner"
            class="flex w-full relative bg-no-repeat bg-cover m-0 p-0 items-center justify-around rounded">
          <input @change="onImageUrlChange($event.target.files[0])" type="file" name="Image" class="hidden">
          <font-awesome-icon v-if="BannerIsHover" icon="camera" size="8x" class="absolute opacity-50" />
        </label>

        <!-- Name -->
        <div class="z-10 w-full py-2 text-white absolute bottom-0 bg-smoke text-center">
          <div
            @mouseover="NameIsHover = true"
            @mouseleave="NameIsHover = false"
            v-if="!isNameEditable" class="flex justify-center"
          >
            <h1 class="text-8xl mx-4 font-semibold tracking-widest">
              {{ nameComputed }}
            </h1>
            <button
              v-show="NameIsHover"
              :disabled="isNameEditable"
              @click="onEditName"
              class="focus:outline-none"
            >
              <font-awesome-icon icon="pen" size="3x" />
            </button>
          </div>
          <div v-else class="">
            <input v-model="Name" type="text" name="Name" class="w-64 text-lg bg-smoke-lightest border-b-4 p-2 mt-2 mb-4 focus:outline-none focus:border-app-tertiary" />
            <editable-field-footer
              v-on:update-editable-state="isNameEditable = false"
              v-on:save="onNameSave" />
          </div>
        </div>
      </section>

      <!-- Genesis -->
      <section class="w-full mt-40 text-center text-app-quinary">
        <div
          @mouseover="GenesisIsHover = true"
          @mouseleave="GenesisIsHover = false"
          class="flex mb-32 justify-center"
        >
          <h1 class="mx-3 text-4xl">GENESIS</h1>
          <button
            v-show="GenesisIsHover"
            :disabled="isCreationEditable"
            @click="onEditCreation"
            class="focus:outline-none"
          >
            <font-awesome-icon icon="pen" />
          </button>
        </div>
        <div v-if="!isCreationEditable" class="flex items-center justify-center flex-col">
          <p v-if="creationComputed === 'UNKNOWN'" class="font-normal leading-relaxed mx-8 mb-8 max-w-lg text-justify">
            Missing data...
          </p>
          <p v-else class="font-normal leading-relaxed mx-8 mb-8 max-w-lg text-justify">
            {{ creationComputed }}
          </p>
        </div>
        <div v-else class="scrollbar">
          <textarea
            v-model="Creation"
            cols="100"
            rows="10"
            class="p-2 bg-app-secondary-light focus:outline-none shadow-lg resize-none force-overflow-content"
          />
          <div class="flex justify-center my-4">
            <editable-field-footer
              v-on:update-editable-state="isCreationEditable = false"
              v-on:save="onCreationSave" />
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script lang="ts">
/* tslint:disable:no-console */

import Vue from 'vue';
import Component from 'vue-class-component';
import { Prop } from 'vue-property-decorator';
import moment from 'moment';

import { join } from 'path';

import Utils from '@/utils';
import { World } from '@/api';

import ProfileBackButton from '@/components/ProfileBackButton.vue';
import EditableFieldFooter from '@/components/EditableFieldFooter.vue';

@Component({
  components: {
    ProfileBackButton,
    EditableFieldFooter,
  },
})
export default class WorldProfile extends Vue {
  @Prop() private Id!: number;

  private utils: Utils = new Utils();

  private world: World = this.$store.getters.getWorldById(this.Id);

  // world properties
  private Name: string = this.world.getName() || '';
  private ImageUrl: string = this.world.getImageUrl() || '';
  private Age: number = this.world.getAge() || 0;
  private Creation: string = this.world.getCreation() || '';

  // isEditable properties
  private isNameEditable: boolean = false;
  private isImageUrlEditable: boolean = false;
  private isAgeEditable: boolean = false;
  private isCreationEditable: boolean = false;
  private BannerIsHover: boolean = false;
  private NameIsHover: boolean = false;
  private GenesisIsHover: boolean = false;

  private destroyed() {
    const filePath: string = join(
      this.utils.getWorldsSubfolderPath(),
      `${this.Id}.json`,
    );
    this.onImageUrlSave();

    this.utils.saveWorld(filePath, this.world);
  }

  // Computed
  private get nameComputed(): string {
    return this.Name.toUpperCase();
  }

  private get createdComputed(): string {
    const timeStamp = this.world.getCreated();
    return moment(timeStamp).calendar();
  }

  private get ageComputed(): number | string {
    return this.Age || 'UNKNOWN';
  }

  private get creationComputed(): string {
    return this.Creation || 'UNKNOWN';
  }

  // onEdit methods
  private onEditName() {
    this.isNameEditable = !this.isNameEditable;
  }

  private onEditImageUrl() {
    this.isImageUrlEditable = !this.isImageUrlEditable;
  }

  private onEditAge() {
    this.isAgeEditable = !this.isAgeEditable;
  }

  private onEditCreation() {
    this.isCreationEditable = !this.isCreationEditable;
  }

  // onSave methods
  private onNameSave() {
    this.world.setName(this.Name);
    this.isNameEditable = !this.isNameEditable;
  }

  private onImageUrlSave() {
    this.world.setImageUrl(this.ImageUrl);
    this.isImageUrlEditable = !this.isImageUrlEditable;
  }

  private onAgeSave() {
    this.world.setAge(this.Age);
    this.isAgeEditable = !this.isAgeEditable;
  }

  private onCreationSave() {
    this.world.setCreation(this.Creation);
    this.isCreationEditable = !this.isCreationEditable;
  }

  // onChange
  private onImageUrlChange(file: any) {
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
}
</script>

<style scoped>

</style>