<template>
  <div>
    <table>
      <thead>
        <tr>
          <th v-for="(label, labelIndex) in labels" :key="labelIndex">
            {{ label.text }}
            <button @click="sortColumn(label.field)">Sort</button>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, itemIndex) in tableData" :key="itemIndex">
          <td v-for="(label, labelIndex) in labels" :key="labelIndex">
            <template v-if="label.field === '_id'">
              {{ itemIndex + 1 }}
            </template>

            <template v-if="label.field === 'picture.thumbnail'">
              <img :src="getNestedValue(item, label.field)" alt="Thumbnail" />
            </template>
            <template v-else-if="label.field === 'dob.date'">
              {{ formattedDOB(getNestedValue(item, label.field)) }}
            </template>
            <template v-else>
              {{ getNestedValue(item, label.field) }}
            </template>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import axios from "axios";

export default {
  setup() {
    const tableData = ref(null);
    const error = ref(null);
    const sortOrder = ref(null);

    const fetchData = async (url) => {
      try {
        const response = await axios.get(url);
        tableData.value = response.data.results;

        console.log(tableData);
      } catch (err) {
        error.value = err;
      }
    };

    onMounted(() => {
      fetchData("https://randomuser.me/api/?results=50");
    });

    const formattedDOB = (timestamp) => {
      const dobDate = new Date(timestamp);
      const options = { day: "numeric", month: "long", year: "numeric" };
      return dobDate.toLocaleDateString("en-US", options);
    };

    const destructureLocation = (location) => {
      const { city, state, country } = location;

      // console.log(city);

      return `${city} ${state} ${country}`;
      // return city;
    };
    const labels = ref([
      { text: "SLNO", field: "_id" },
      { text: "Name", field: "name.first" },
      { text: "DOB", field: "dob.date" },
      { text: "Email", field: "email" },
      { text: "Location", field: "location.city" },
      { text: "Phone", field: "phone" },
      { text: "Picture", field: "picture.thumbnail" },
    ]);

    const getNestedValue = (obj, path) => {
      const keys = path.split(".");
      return keys.reduce((acc, key) => (acc ? acc[key] : ""), obj);
    };

    const sortColumn = (field) => {
      if (!sortOrder.hasOwnProperty(field)) {
      
        sortOrder[field] = "asc";
      } else {
        sortOrder[field] = sortOrder[field] === "asc" ? "desc" : "asc";
      }

      tableData.value.sort((a, b) => {
        const valueA = getNestedValue(a, field);
        const valueB = getNestedValue(b, field);

        if (typeof valueA === "string" && typeof valueB === "string") {
          return sortOrder[field] === "asc"
            ? valueA.localeCompare(valueB)
            : valueB.localeCompare(valueA);
        } else {
          return sortOrder[field] === "asc" ? valueA - valueB : valueB - valueA;
        }
      });
    };
    return {
      tableData,
      error,
      formattedDOB,
      destructureLocation,
      labels,
      getNestedValue,
      sortColumn,
    };
  },
};
</script>
<style>
table {
  border-collapse: collapse;
  width: 100%;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  overflow: hidden;
}

th,
td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #4caf50;
  color: #fff;
}

tr:hover {
  background-color: #f5f5f5;
}

tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}
</style>
