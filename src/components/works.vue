<script setup>
import { ref, computed } from "vue"
const props = defineProps({
   rezumes: Array,
   reviews: Array,
})
const emit = defineEmits(['actionSix'])

const filterPortfolio = ref('all') // 'all', 'yes', 'no'
const minRating = ref(0)

function connect(author){
   emit('actionSix', author)
}

const staticResumes = ref([
  {
    skills: 'Пишу текста для постов, визитных карточек, описаний товаров',
    pay: 'До 300 слов: 200р. От 300 слов: 500р.',
    portfolio: 'Нет',
    author: 'Нэнси',
    rating: 4.2
  },
  {
    skills: 'Делаю курсовики и дипломники',
    pay: 'На 3: 1500р. На 4: 2000р. На 5: 2500р.',
    portfolio: 'Есть',
    author: 'Сэм',
    rating: 3.8
  },
  {
    skills: 'Пишу художественные текста, фанфики, сочинения',
    pay: 'До 500 слов: 300р. До 1000 слов: 500р. От 500 слов: 1000р.',
    portfolio: 'Есть',
    author: 'Йозеф',
    rating: 4.7
  }
])

// Вычисляем рейтинг для каждого копирайтера
const resumesWithRating = computed(() => {
  const dynamicResumes = props.rezumes.map(resume => {
    const copywriterReviews = props.reviews?.filter(r => r.copywriter === resume.author) || []
    const rating = copywriterReviews.length > 0 
      ? (copywriterReviews.reduce((sum, r) => sum + r.rating, 0) / copywriterReviews.length).toFixed(1)
      : null
      
    return {
      ...resume,
      rating,
      hasPortfolio: resume.portfolio && resume.portfolio.trim() !== '' && resume.portfolio.toLowerCase() !== 'нет'
    }
  })
  
  return [...staticResumes.value, ...dynamicResumes]
})

// Фильтруем резюме
const filteredResumes = computed(() => {
  return resumesWithRating.value.filter(resume => {
    // Фильтр по портфолио
    const portfolioMatch = 
      filterPortfolio.value === 'all' ||
      (filterPortfolio.value === 'yes' && resume.hasPortfolio) ||
      (filterPortfolio.value === 'no' && !resume.hasPortfolio)
    
    // Фильтр по рейтингу
    const ratingMatch = 
      minRating.value === 0 || 
      (resume.rating && parseFloat(resume.rating) >= minRating.value)
    
    return portfolioMatch && ratingMatch
  })
})
</script>
<template>
<div class="filters">
  <div class="filter-group">
    <label>Портфолио:</label>
    <select v-model="filterPortfolio">
      <option value="all">Все</option>
      <option value="yes">Есть портфолио</option>
      <option value="no">Нет портфолио</option>
    </select>
  </div>
  
  <div class="filter-group">
    <label>Минимальный рейтинг:</label>
    <select v-model="minRating">
      <option :value="0">Любой</option>
      <option :value="3">3+</option>
      <option :value="4">4+</option>
      <option :value="4.5">4.5+</option>
    </select>
  </div>
</div>

<div class="gridd">
  <div v-for="(item, index) in filteredResumes" :key="index" class="square"> 
    <div class="card-content">
      <p>Навыки и умения: {{ item.skills }}</p> 
      <p>Расценки: {{ item.pay }}</p> 
      <p>Портфолио: {{ item.portfolio }}</p>
      <p>Автор: {{ item.author }}</p>
      <div class="rating-container">
        <p v-if="item.rating !== null">Рейтинг: {{ item.rating }} ★</p>
        <p v-else>Рейтинг: нет оценок</p>
      </div>
    </div>
    <button @click="connect(item.author)" class="btn">Связаться с автором</button>
  </div> 
</div>
</template>
<style scoped>
.gridd{
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 10px; 
      padding: 20px;
}
.square {
  position: relative;
  width: 270px;
  min-height: 300px;
    height: auto;
    color: black;
    background: linear-gradient(to bottom left, rgba(183, 45, 45, 0.5), rgb(250, 167, 181, 0.5));
    border-radius: 25%;
    padding: 20px;
    display: flex;
      flex-direction: column;
  justify-content: space-between;
    margin-top: 50px;
    word-wrap: break-word; 
    white-space: normal; 
}

.card-content {
  flex-grow: 1;
}

.rating-container {
  margin: 15px 0;
}
.filters {
  display: flex;
  
  gap: 20px;
  margin: 20px 0;
  margin-top: 100px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 10px;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.filter-group label {
  font-weight: bold;
}

.filter-group select {
  padding: 5px 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.square p:last-of-type {
  margin-bottom: 50px; 
}

 .btn {
      position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: calc(100% - 100px);
  margin-top: 10px;
   height: auto;
   background: linear-gradient(to bottom right, rgb(255, 132, 0), rgb(250, 167, 181));
   border-color: rgb(202, 43, 11);
}

.btn:hover{
   background: linear-gradient(to bottom right, rgb(196, 88, 0), rgb(183, 108, 108));
}
</style>