<template>
  <div>
    <!-- 職業選擇 -->
    <label>選擇職業:
      <select v-model="playerClass">
        <option v-for="Class in totalClass" :key="Class" :value="Class">{{ Class }}</option>
      </select>
    </label>

    <!-- 角色等級輸入框 -->
    <label>職業等級: <input type="number" v-model.number="playerLevel" :min="0" :max="130"/></label>

    <AttributePoint @updateStats="updateStats" />
    <AttributeItem @updateStats="updateStats" />
    <TotalStats :stats="totalStats" />
  </div>
</template>

<script>
import AttributePoint from "./components/AttributePoint.vue";
import AttributeItem from "./components/AttributeItem.vue";
import TotalStats from "./components/TotalStats.vue";

export default {
  components: {
    AttributePoint,
    AttributeItem,
    TotalStats
  },
  data() {
    return {
      playerLevel: 1, // 玩家等級
      playerClass: "none", // 玩家職業
      totalClass: {}, // 所有職業
      totalStats: {} // 存放計算後的總屬性
    };
  },
  mounted() {
    this.loadBaseStats(); // 載入基礎屬性
    this.loadClassList(); // 載入職業列表
  },
  methods: {
    // 讀取 `class.json`
    loadClassList() {
      fetch("/class.json")
        .then(response => response.json())
        .then(data => {
          // 轉換 class.json 陣列為物件格式
          this.totalStats = data.reduce((acc, attr) => {
            acc[attr.name] = { value: attr.value, suffix: attr.suffix };
            return acc;
          }, {});
          this.playerClass = this.totalStats[0]; // 預設選擇第一個職業
        })
        //.catch(error => console.error("職業列表載入錯誤:", error));
    },
    // 讀取 base.json，設定基礎屬性
    loadBaseStats() {
      fetch("/base.json")
        .then(response => response.json())
        .then(data => {
          // 轉換 base.json 陣列為物件格式
          this.totalStats = data.reduce((acc, attr) => {
            acc[attr.name] = { value: attr.value, suffix: attr.suffix };
            return acc;
          }, {});
        })
        .catch(error => console.error("基礎數據載入錯誤:", error));
    },

    // 當道具影響數值時，更新總屬性
    updateStats(differenceStats) {
      for (const key in differenceStats) {
      if (this.totalStats[key]) {
          this.totalStats[key].value += differenceStats[key];
        }
      }
    }
  }
};
</script>
