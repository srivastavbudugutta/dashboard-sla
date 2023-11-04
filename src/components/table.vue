<template>
  <div>
    <header>
      <!-- Hide By Status Bar -->
      <div class="hideBar" style="padding: 10px;width: 100%;background: cadetblue;">
        <label class="hideLabel">Hide:</label>
        <div class="checkbox">
          <!-- All status Checkbox -->
          <input
            id="allStatus"
            type="checkbox"
            class="styled"
            @change="toggleAllStatuses"
            v-model="hideAllStatus"
          />
          <!-- Dynamic Status Checkboxes -->
          <label for="allStatus">All statuses:</label>
          <div v-for="status in productDataBystatus.status" :key="status" >
            
            <label :for="`status-${status}`">{{ status }}</label>
            <input
              :id="`status-${status}`"
              type="checkbox"
              class="styled"
              :value="status"
              v-model="hidestatus"
            />
            
          </div>
        </div>
      </div>
    </header>

    <!-- Main Table Design -->
    <body style="max-height: calc(100vh - 100px); overflow-y: auto;">
      <table>
        <div>
          <thead style="background: aliceblue;">
            <tr>
              <td :colspan="12">Dashboard SLA</td>
            </tr>
            <tr>
              <th colspan="3">{{ wwData }}</th>
              <th colspan="8">Product Info</th>
            </tr>
            <tr>
              <th>Status</th>
              <th>Cores</th>
              <th class="width1">Product</th>
              <th class="width1">Lithography</th>
              <th>Threads</th>
              <th>Base Freq</th>
              <th>Max Turbo Freq</th>
            </tr>
          </thead>
        
          <tbody>
            <template v-for="(data, status) in paginatedData" :key="status">
              <!-- status -->
              <tr :class="getRowColorClass(status)">
                <td class="width1" :rowspan="calstatusRowspan(data)">
                  {{ status }}
                </td>
              </tr>

              <template v-for="cores in Object.keys(data)">
                <!-- cores -->
                <tr>
                  <td class="width1" :rowspan="Object.keys(data[cores]).length + 1">
                    {{ cores }}
                  </td>
                </tr>

                <tr v-for="(v, k) in data[cores]">
                  <!-- product -->
                  <td class="productColumn">{{ v.Product }}</td>

                  <!-- Lithography -->
                  <td>{{ v.Lithography }}</td>

                  <!-- Threads -->
                  <td>
                    <div class="innerCells">
                      <input :value="v.Threads" :disabled="true" type="text" />
                    </div>
                  </td>

                  <!-- Base Freq -->
                  <td>
                    <div class="innerCells">
                      <input :value="v.Base_Freq" :disabled="true" type="text" />
                    </div>
                  </td>

                  <!-- Max Turbo Freq -->
                  <td>
                    <div class="innerCells">
                      <input :value="v.Max_Turbo_Freq" type="text" :disabled="true" />
                    </div>
                  </td>
                </tr>
              </template>
            </template>
          </tbody>
        </div>
      </table>
    </body>

    <footer>
      <!-- Pagination Controls -->
      <div class="pagination-controls">
        <button @click="changePage(1)" :disabled="currentPage === 1">First</button>
        <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">Previous</button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button @click="changePage(currentPage + 1)" :disabled="currentPage >= totalPages">Next</button>
        <button @click="changePage(totalPages)" :disabled="currentPage === totalPages">Last</button>
      </div>
    </footer>

    
  </div>
</template>

<script>
import { onMounted, ref, computed } from 'vue';
import data from "../assets/data.json";

export default {
  setup() {
    const currentPage = ref(1);
    const rowsPerPage = 100;
    const totalRows = computed(() => {
      return Object.values(productDataBystatus.value.data).reduce(
        (acc, statusData) => acc + Object.values(statusData).reduce(
          (innerAcc, coreData) => innerAcc + coreData.length, 0
        ), 0
      );
    });
    const totalPages = computed(() => Math.ceil(totalRows.value / rowsPerPage));

    const paginatedData = computed(() => {
      const start = (currentPage.value - 1) * rowsPerPage;
      let end = start + rowsPerPage;
      let paginated = {};
      let count = 0;

      for (const [status, coresData] of Object.entries(productDataBystatus.value.data)) {
        for (const [core, products] of Object.entries(coresData)) {
          for (const product of products) {
            if (count >= start && count < end) {
              if (!paginated[status]) paginated[status] = {};
              if (!paginated[status][core]) paginated[status][core] = [];
              paginated[status][core].push(product);
            }
            count++;
            if (count >= end) break;
          }
          if (count >= end) break;
        }
        if (count >= end) break;
      }

      return paginated;
    });

    const changePage = (newPage) => {
      currentPage.value = newPage;
    };

    const getRowColorClass = (status) => {
      console.log("The status",status)
      
      switch (status) {
        case 'Launched': return 'launched';
        case 'Discontinued': return 'discontinued';
        case 'Announced': return 'announced';
        case 'Launched (with IPU)': return 'IPU';
        default: return '';
      }
    };
    const hidestatus = ref([]);
    const hideAllStatus = ref(false);
    const UIData = ref(data);
    const wwInfo = ref({});

    onMounted(() => {
      wwInfo.value = getWWFromDate();
    });

    const wwData = computed(() => `${wwInfo.value.year}WW${wwInfo.value.workweek}.${wwInfo.value.numofday}`);

    const productDataBystatus = computed(() => {
      let tmp = {};
      let statusSet = new Set();

      UIData.value.forEach((element) => {
        let status = element.Status;
        let cores = element.Cores;

        // push status to set
        statusSet.add(status);

        if (hidestatus.value.includes(status)) return; // Hide by status
        if (!tmp[status]) tmp[status] = {};
        if (!tmp[status][cores]) tmp[status][cores] = [];

        tmp[status][cores].push(element);
      });

      // sort status in order
      const strings = new Set(statusSet);
      const sortedStringsArray = [...strings].sort();
      statusSet = new Set(sortedStringsArray);
      console.log("The status data", statusSet)
      return {
        status: [...statusSet],
        data: tmp,
      };
    });

    const calstatusRowspan = (data) => {
      let sum = Object.keys(data).length + 1;
      for (const cores in data) {
        sum += Object.keys(data[cores]).length;
      }
      return sum;
    };

    const getWWFromDate = (date = new Date()) => {
      let currentDate = date || new Date();
      let startDate = new Date(currentDate.getFullYear(), 0, 1);
      let days = Math.floor((currentDate - startDate) / (24 * 60 * 60 * 1000));

      return {
        year: currentDate.getFullYear(),
        workweek: Math.ceil(days / 7),
        numofday: currentDate.getDay(),
      };
    };

    const toggleAllStatuses = () => {
      if (hideAllStatus.value) {
        hidestatus.value = productDataBystatus.value.status;
      } else {
        hidestatus.value = [];
      }
    };

    return {
      hidestatus,
      hideAllStatus,
      wwData,
      productDataBystatus,
      calstatusRowspan,
      toggleAllStatuses,
      currentPage,
      totalPages,
      paginatedData,
      changePage,
      getRowColorClass
    };
  }
};
</script>


<style scoped>
.pagination-controls {
  display: flex;
  justify-content: center;
  margin-top: 10px;
}

.pagination-controls button {
  margin: 0 5px;
  padding: 5px 10px;
}
.launched {
  background-color: #7fff00; /* Green for launched */
}

.discontinued {
  background-color: #82ffac; /* Light Green for discontinued */
}

.announced {
  background-color: #ffa500; /* Orange for announced */
}

.IPU {
  background-color: #dcdcdc; /* Grey for IPU */
}
.fas.fa-times {
  display: none;
}

.fas.fa-times.comment {
  display: block;
}

.overWrittenCells:hover .fas {
  display: block;
}

.innerCells {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}

.innerCells.comment {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 15px;
}

table {
  width: 100%;
  white-space: nowrap !important;
}

table td {
  position: relative;
}

i {
  cursor: pointer;
}

.legendColorBox {
  margin: 0.4%;
  float: left;
  height: 20px;
  width: 30px;
  border: 1px solid grey;
  margin-right: 4%;
}

.inputBox {
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  text-align: center;
  border: 0;
  text-transform: uppercase !important;
}

.inputBoxOverWritten {
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  text-align: center;
  border: 0;
  width: 80px;
  text-transform: uppercase !important;
  background: none !important;
}

.overWrittenCells {
  border: 2px solid rgb(194, 1, 1);
}

.overWrittenCells input {
  outline: 0;
}

input::placeholder {
  color: black;
}

input:focus::-webkit-input-placeholder {
  color: grey;
}

input[disabled] {
  cursor: text;
  background-color: inherit;
  color: black;
}

.legend-labels {
  white-space: nowrap !important;
  padding: 0%;
  display: flex;
  list-style-type: none;
  margin-bottom: 5px;
}

.legend-labels li {
  font-size: small;
  margin-right: 2%;
}

select {
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  text-align: center;
  border: 0;
}

table tr td:not(.skip),
table tr th {
  text-align: center;
}

td,
th {
  padding: 2px !important;
  width: 100px;
  border: 1px solid black;
}

.launched {
  width: 1%;
  background-color: #7fff00;
}

.announced {
  width: 1%;
  background-color: #ffa500;
}

.IPU {
  width: 1%;
  background-color: #dcdcdc;
}

.discontinued {
  width: 1%;
  background-color: #82ffac;
}

.hideBar {
  list-style: none;
  display: flex;
}

.productColumn {
  width: 1%;
  background-color: white;
}

.checkbox {
  list-style: none;
  display: flex;
}

.checkbox label {
  margin-left: 10px;
}

.redActual {
  width: 1%;
  color: red;
  background-color: #dcdcdc;
}

.width1 {
  width: 1%;
  /* white-space: nowrap !important; */
}
</style>