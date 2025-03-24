<template>
  <div>
    <p></p>

    <table border="1">
      <thead>
        <tr>
          <th>屬性道具</th>
          <th>數量</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in props" :key="item.name">
          <td>{{ item.name }}</td>
          <td>
            <input 
              type="number" 
              class="attribute_item-input"
              v-model.number="item.used" 
              :min="0" 
              :max="item.maxUses"
              @input="calculateAttributes"
            />
          </td>
        </tr>
      </tbody>
    </table>
    <button @click="resetUses">重置屬性道具</button>
    <button @click="maxUses">吃滿屬性道具</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      props: [], // 屬性道具列表
      previousStats: {} // 前次屬性總和
    };
  },
  mounted() {
    this.loadAttributeItems();
  },
  methods: {
    // 讀取 attribute_item.json
    loadAttributeItems() {
      fetch("/attribute_item.json")
        .then(response => {
          return response.json();
        })
        .then(data => {
          this.props = data.map(item => ({ ...item, used: 0 })); // 新增 `used` 屬性
        })
        .catch(error => console.error("屬性道具數據載入錯誤:", error));
    },

    // 計算屬性影響並傳遞給 `App.vue`
    calculateAttributes() {
      let newStats = {};
      this.props.forEach(item => {
        item.effect.forEach(effect => {
          if (!newStats[effect.attributes]) {
            newStats[effect.attributes] = 0;
          }
          newStats[effect.attributes] += effect.value * item.used;
        });
      });
       // 計算變化量 (newStats - previousStats)
       let differenceStats = {};
      for (const key in newStats) {
        differenceStats[key] = newStats[key] - (this.previousStats[key] || 0);
      };

      // **確保 previousStats 也包含新出現的屬性**
      for (const key in this.previousStats) {
        if (!(key in newStats)) {
          differenceStats[key] = -this.previousStats[key]; // 若屬性已消失，則減去原數值
        }
      };

      // 更新 previousStats，讓下一次比較用
      this.previousStats = { ...newStats };

      // 回傳變化量給 App.vue
      this.$emit("updateStats", differenceStats);
    },
      

    // JSON 上傳處理
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.readAsText(file);
      reader.onload = () => {
        try {
          const uploadedData = JSON.parse(reader.result);
          this.updateUsage(uploadedData);
        } catch (error) {
          console.error("上傳 JSON 解析錯誤:", error);
        }
      };
    },

    // 更新使用次數
    updateUsage(uploadedData) {
      uploadedData.forEach(uploadedItem => {
        let prop = this.props.find(item => item.name === uploadedItem.name);
        if (prop) {
          prop.used = Math.min(uploadedItem.used, prop.maxUses); // 限制最大使用次數
        }
      });

      this.calculateAttributes(); // 重新計算總屬性
    },

    // 重置屬性道具
    resetUses() {
      this.props.forEach(item => (item.used = 0));
      this.calculateAttributes();
    },

    // 吃滿屬性道具
    maxUses() {
      this.props.forEach(item => (item.used = item.maxUses));
      this.calculateAttributes();
    }
  }
};
</script>