<template>
  <div>
    <h2>總屬性</h2>
    <table border="1">
      <tr>
        <th>屬性名稱</th>
        <th>數值</th>
      </tr>
      <tr v-for="(stat, name) in stats" :key="name">
        <td>{{ name }}</td>
        <td>{{ stat.value }}{{ stat.suffix }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  props: ["stats"]
};
</script>