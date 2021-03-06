<template>
  <div :class="`container__modal  ${!visible && 'container__modal--hidden'}`">
    <div class="close" @click="changeVisibility()">
      <font-awesome-icon icon="times" color="#FFF" size="lg"/>
    </div>
    <div class="modal">
      <div class="title">
        <h2 class="text--m">Adicionar bolsa</h2>
        <p class="text--s">Filtre e adicione as bolsas de seu interesse.</p>
      </div>
      <div class="filter">
        <div class="filter__top">
          <div class="filter__top__item">
            <label class="text--xs">SELECIONE SUA CIDADE</label>
            <select class="select" v-on:change="e => filterScholarships('city', e.target.value)">
              <option>Todos</option>
              <option v-for="(city, index) in cities" :key="index">{{ city }}</option>
            </select>
          </div>
          <div class="filter__top__item">
            <label class="text--xs">SELECIONE O CURSO DE SUA PREFERÊNCIA</label>
            <select class="select" v-on:change="e => filterScholarships('course', e.target.value)">
              <option>Todos</option>
              <option v-for="(course, index) in courses" :key="index">{{ course }}</option>
            </select>
          </div>
        </div>
        <div class="filter__bottom">
          <div class="filter__bottom__type">
            <div>
              <label class="text--xs">COMO VOCÊ QUER ESTUDAR?</label>
            </div>
            <div class="filter__bottom__type__options">
              <div
                class="filter__bottom__type__options__checkbox"
                @click="filterScholarships('presential', !filter.presential)"
              >
                <input class="checkbox" type="checkbox">
                <span class="text--s">Presencial</span>
              </div>
              <div
                class="filter__bottom__type__options__checkbox"
                @click="filterScholarships('ead', !filter.ead)"
              >
                <input class="checkbox" type="checkbox">
                <span class="text--s">A distância</span>
              </div>
            </div>
          </div>
          <div class="filter__bottom__price">
            <div class="filter__bottom__price__options">
              <p class="text--xs">ATÉ QUANTO PODE PAGAR?</p>
              <p class="text--xs">R$ 10.000</p>
            </div>
            <div>
              <VueSlider
                :dotSize="25"
                :max="10000"
                :interval="100"
                :value="0"
                @change="value => filterScholarships('price', value)"
              />
            </div>
          </div>
        </div>
      </div>
      <div class="sort">
        <div class="sort__right">
          <p class="text--s">Resultado:</p>
        </div>
        <div class="sort__left">
          <p class="text--s">Ordenar por:</p>
          <select class="select select--text text--s">
            <option class="text--s">NOME DA FACULDADE</option>
          </select>
        </div>
      </div>
      <div class="results">
        <NewScholarshipItem
          :key="scholarship.id"
          v-for="scholarship in filteredScholarships"
          :item="scholarship"
          v-on:selectItem="selectItem"
        />
      </div>
      <div class="buttons">
        <button
          @click="changeVisibility"
          class="buttons__item buttons__item--outline"
          name="cancel"
        >Cancelar</button>
        <button
          v-if="selectedScholarships.length === 0"
          class="buttons__item buttons__item--disabled"
          name="add"
        >Adicionar bolsa(s)</button>
        <button
          v-else
          @click="addFavorites"
          class="buttons__item buttons__item--yellow"
          name="add"
        >Adicionar bolsa(s)</button>
      </div>
    </div>
  </div>
</template>
<style lang="scss" scoped>
@import "./_newScholarship.scss";
</style>



<script>
import NewScholarshipItem from "./newScholarshipItem/NewScholarshipItem.vue";
import VueSlider from "vue-slider-component";
import { sortArray } from "../../assets/utils";
import "vue-slider-component/theme/antd.css";
export default {
  name: "newScholarship",
  components: {
    NewScholarshipItem,
    VueSlider
  },
  data() {
    return {
      visible: false,
      scholarships: [],
      filteredScholarships: [],
      selectedScholarships: [],
      favoriteScholarships: [],
      cities: [],
      courses: [],
      filter: {
        city: "TODOS",
        course: "TODOS",
        ead: false,
        presential: false,
        price: 0
      }
    };
  },
  methods: {
    changeVisibility() {
      this.visible = !this.visible;
    },
    setScholarships(scholarships) {
      this.scholarships = sortArray(scholarships, "university.name");
      this.filteredScholarships = this.scholarships;
      this.cities = [...new Set(scholarships.map(item => item.campus.city))];
      this.courses = [...new Set(scholarships.map(item => item.course.name))];
    },
    filterScholarships(prop, value) {
      this.filter[prop] = value;
      this.filteredScholarships = this.scholarships.filter(this.filterRules);
    },
    filterRules(item) {
      const { city, course, ead, presential, price } = this.filter;

      if (city && city !== item.campus.city && city.toUpperCase() !== "TODOS") {
        return false;
      }
      if (
        course &&
        course !== item.course.name &&
        course.toUpperCase() !== "TODOS"
      ) {
        return false;
      }

      //SE AS DUAS OPÇÕES ESTIVEREM HABILITADAS ENTÃO NÃO FILTRA
      if (!presential || !ead) {
        if (ead && item.course.kind.toUpperCase() !== "EAD") {
          return false;
        }

        if (presential && item.course.kind.toUpperCase() !== "PRESENCIAL") {
          return false;
        }
      }

      if (price >= item.price_with_discount) {
        return false;
      }

      return true;
    },
    selectItem(id) {
      if (this.selectedScholarships.filter(item => item === id) > 0) {
        this.selectedScholarships = this.selectedScholarships.filter(
          item => item !== id
        );
      } else {
        this.selectedScholarships = [id, ...this.selectedScholarships];
      }
    },
    addFavorites() {
      this.$emit("addFavorites", [
        this.selectedScholarships,
        this.scholarships
      ]);
    }
  }
};
</script>
