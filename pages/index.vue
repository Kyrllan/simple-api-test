<template>
  <v-container id="root" class="pt-4">
    <v-row no-gutters>
      <v-col cols="12" class="d-flex">
        <v-select
          dense
          outlined
          v-model="methodSelected"
          :items="methods"
          style="max-width: 140px"
          label="Method"
          item-text="text"
          item-value="value"
          return-object
        >
          <template v-slot:selection="{ item }">
            <span :style="`color: ${item.color}`">
              {{ item.text }}
            </span>
          </template>
          <template v-slot:item="{ item }">
            <span :style="`color: ${item.color}`">
              {{ item.text }}
            </span>
          </template>
        </v-select>
        <v-text-field
          class="pl-2"
          dense
          outlined
          persistent-placeholder
          v-model="url"
          clearable
          label="URL"
        ></v-text-field>
        <div style="width: 40%">
          <v-text-field
            class="pl-2"
            dense
            outlined
            persistent-placeholder
            v-model="token"
            clearable
            :disabled="!auth"
            label="Bearer token"
          ></v-text-field>
          <v-checkbox
            v-model="auth"
            label="Authorization"
            class="pl-2"
            style="margin-top: -25px"
          ></v-checkbox>
        </div>
        <v-btn
          class="ml-2"
          color="#28A745"
          style="height: 40px; width: 100px"
          :loading="loading"
          @click="send"
          >Send</v-btn
        >
        <v-btn class="ml-2" color="#DC3545" style="height: 40px; width: 100px"
          >Cancel</v-btn
        >
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              class="ml-2 mt-1"
              icon
              v-bind="attrs"
              v-on="on"
              @click="clearData()"
            >
              <v-icon color="#4FC3F7">mdi-broom</v-icon>
            </v-btn>
          </template>
          <span>Clean</span>
        </v-tooltip>
      </v-col>
    </v-row>
    <v-row no-gutters class="no-selectable">
      <v-col cols="12" class="d-flex">
        <v-spacer />
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <div
              class="chip-status"
              :class="statusColor(status)"
              v-bind="attrs"
              v-on="on"
            >
              {{ status }}
            </div>
          </template>
          <span>Status code</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <div class="chip" v-bind="attrs" v-on="on">
              {{ timeResponse.seconds }},{{ timeResponse.milliseconds }} ms
            </div>
          </template>
          <span>Request time</span>
        </v-tooltip>
      </v-col>
    </v-row>
    <v-row no-gutters>
      <v-col cols="12">
        <div class="parent">
          <div class="div1 mt-2">
            <v-card elevation="0" color="rgba(31, 41, 55, 1)">
              <div class="pl-2 py-2 no-selectable">
                <span>Body</span>
              </div>
              <v-divider></v-divider>
              <div style="overflow: auto; height: 68vh">
                <textarea
                  class="ma-1 pa-1"
                  style="
                    resize: none;
                    height: 97%;
                    width: 96%;
                    background-color: transparent;
                    color: white;
                  "
                  v-model="bodyJson"
                >
                </textarea>
              </div>
            </v-card>
          </div>
          <div class="div2 mt-2">
            <v-card elevation="0" color="rgba(31, 41, 55, 1)">
              <div class="pl-2 py-2 no-selectable">
                <span>Data</span>
              </div>
              <v-divider></v-divider>
              <div style="overflow: auto; height: 68vh">
                <pre
                  style="background-color: transparent; height: 66vh"
                ><code id="data" class="language-js line-numbers" style="background-color: transparent">
            </code></pre>
              </div>
            </v-card>
          </div>
          <div class="div3 mt-2">
            <v-card elevation="0" color="rgba(31, 41, 55, 1)">
              <div class="pl-2 py-2 no-selectable">
                <span>Headers</span>
              </div>
              <v-divider></v-divider>
              <div style="height: 31vh">
                <pre
                  style="background-color: transparent; height: 31vh"
                ><code id="headers" class="language-js line-numbers" style="background-color: transparent">
            </code></pre>
              </div>
            </v-card>
          </div>
          <div class="div4 mt-1">
            <v-card elevation="0" color="rgba(31, 41, 55, 1)">
              <div class="pl-2 py-2 no-selectable">
                <span>Config</span>
              </div>
              <v-divider></v-divider>
              <div style="overflow: auto; height: 30vh">
                <pre
                  style="background-color: transparent"
                ><code id="config" class="language-js line-numbers" style="background-color: transparent">
            </code></pre>
              </div>
            </v-card>
          </div>
        </div>
      </v-col>
    </v-row>
    <v-snackbar v-model="snackbar" :timeout="3000" color="red accent-2" right>
      {{ snackbarText }}
    </v-snackbar>
  </v-container>
</template>

<script>
import axios from "axios";
import Prism from "~/plugins/prism";

export default {
  name: "IndexPage",
  data: () => ({
    snackbar: false,
    snackbarText: "",
    methodSelected: { value: "get", text: "GET" },
    methods: [
      { value: "get", text: "GET", color: "#6D28D9" },
      { value: "post", text: "POST", color: "#4CAF50" },
      { value: "put", text: "PUT", color: "#B45309" },
      { value: "patch", text: "PATCH", color: "#FFCC80" },
      { value: "delete", text: "DELETE", color: "#B91C1C" },
    ],
    url: "",
    bodyJson: "",
    loading: false,
    token: "",
    auth: false,
    status: null,
    timeResponse: {
      seconds: 0,
      milliseconds: 0,
    },
    stopwatchTimer: null,
    milliseconds: 0,
    seconds: 0,
    dataEl: null,
    headersEl: null,
    configEl: null,
  }),
  mounted() {
    this.dataEl = document.getElementById("data");
    this.headersEl = document.getElementById("headers");
    this.configEl = document.getElementById("config");
  },
  watch: {
    auth() {
      if (!this.auth) {
        this.token = "";
      }
    },
  },
  methods: {
    statusColor(status) {
      if (status >= 500) {
        return "status500";
      } else if (status >= 400) {
        return "status400";
      } else if (status >= 300) {
        return "status300";
      } else if (status >= 200) {
        return "status200";
      }
      return "status-default";
    },
    send() {
      switch (this.methodSelected.value) {
        case "get":
          this.get();
          break;
        case "post":
          this.post();
          break;
        case "put":
          this.put();
          break;
        case "patch":
          this.patch();
          break;
        case "delete":
          this.delete();
          break;
        default:
          break;
      }
    },
    async get() {
      this.loading = true;
      this.startTimer();
      try {
        const response = await axios.get(this.url);
        this.renderOutput(response);
      } catch (error) {
        this.renderError(error);
      } finally {
        this.loading = false;
        this.pauseTimer();
      }
    },
    async post() {
      this.loading = true;
      this.startTimer();
      try {
        const response = await axios.post(this.url, JSON.parse(this.bodyJson));
        this.renderOutput(response);
      } catch (error) {
        this.renderError(error);
      } finally {
        this.loading = false;
        this.pauseTimer();
      }
    },
    async put() {
      this.loading = true;
      this.startTimer();
      try {
        const response = await axios.put(this.url, JSON.parse(this.bodyJson));
        this.renderOutput(response);
      } catch (error) {
        this.renderError(error);
      } finally {
        this.loading = false;
        this.pauseTimer();
      }
    },
    async patch() {
      this.loading = true;
      this.startTimer();
      try {
        const response = await axios.patch(this.url, JSON.parse(this.bodyJson));
        this.renderOutput(response);
      } catch (error) {
        this.renderError(error);
      } finally {
        this.loading = false;
        this.pauseTimer();
      }
    },
    async delete() {
      this.loading = true;
      this.startTimer();
      try {
        const response = await axios.delete(this.url, this.bodyJson ? JSON.parse(this.bodyJson) : '');
        this.renderOutput(response);
      } catch (error) {
        this.renderError(error);
      } finally {
        this.loading = false;
        this.pauseTimer();
      }
    },
    renderOutput(response) {
      this.status = response.status;
      // Data
      this.dataEl.innerHTML = JSON.stringify(response.data, null, 2);
      Prism.highlightElement(this.dataEl);
      // Headers
      this.headersEl.innerHTML = JSON.stringify(response.headers, null, 2);
      Prism.highlightElement(this.headersEl);
      // Config
      this.configEl.innerHTML = JSON.stringify(response.config, null, 2);
      Prism.highlightElement(this.configEl);
    },
    renderError(error) {
      this.snackbar = true;
      this.snackbarText = error;
      this.status = error.response.status;
      // Data
      this.dataEl.innerHTML = JSON.stringify(
        this.statusCodeErrorMessage(this.status),
        null,
        2
      );
      Prism.highlightElement(this.dataEl);
      // Headers
      this.headersEl.innerHTML = JSON.stringify(
        error.response.headers,
        null,
        2
      );
      Prism.highlightElement(this.headersEl);
      // Config
      this.configEl.innerHTML = JSON.stringify(error.response.config, null, 2);
      Prism.highlightElement(this.configEl);
    },
    startTimer() {
      this.stopTimer();
      if (this.stopwatchTimer) return;
      this.stopwatchTimer = setInterval(() => {
        this.timeResponse.milliseconds = this.milliseconds
          .toString()
          .slice(0, 2);
        this.milliseconds += 10;

        if (this.seconds === 59 && this.milliseconds === 990) {
          stopInterval();
        } else if (this.milliseconds === 1000) {
          this.milliseconds = 0;
          this.seconds++;
          this.timeResponse.seconds = this.seconds;
        }
      }, 10);
    },
    stopTimer() {
      clearInterval(this.stopwatchTimer);
      this.stopwatchTimer = null;
      this.milliseconds = 0;
      this.seconds = 0;
      this.timeResponse.milliseconds = "00";
      this.timeResponse.seconds = this.seconds;
    },
    pauseTimer() {
      clearInterval(this.stopwatchTimer);
      this.stopwatchTimer = null;
    },
    clearData() {
      this.dataEl.innerHTML = "";
      this.headersEl.innerHTML = "";
      this.configEl.innerHTML = "";
      this.bodyJson = ''
    },
    statusCodeErrorMessage(status) {
      switch (status) {
        case 400:
          return "Bad Request";
        case 401:
          return "Unauthorized";
        case 402:
          return "Payment Required";
        case 403:
          return "Forbidden";
        case 404:
          return "Not Found";
        case 405:
          return "Method Not Allowed";
        case 406:
          return "Not Acceptable";
        case 407:
          return "Proxy Authentication Required";
        case 408:
          return "Request Timeout";
        case 409:
          return "Conflict";
        case 410:
          return "Gone";
        case 411:
          return "Length Required";
        case 412:
          return "Precondition Failed";
        case 413:
          return "Payload Too Large";
        case 414:
          return "URI Too Long";
        case 415:
          return "Unsupported Media Type";
        case 416:
          return "Range Not Satisfiable";
        case 417:
          return "Expectation Failed";
        case 418:
          return "Im a teapot";
        case 421:
          return "Misdirected Request";
        case 422:
          return "Unprocessable Entity";
        case 423:
          return "Locked";
        case 424:
          return "Failed Dependency";
        case 425:
          return "Too Early";
        case 426:
          return "Upgrade Required";
        case 428:
          return "Precondition Required";
        case 429:
          return "Too Many Requests";
        case 431:
          return "Request Header Fields Too Large";
        case 451:
          return "Unavailable For Legal Reasons ";
        case 500:
          return "Internal Server Error";
        case 501:
          return "Not Implemented";
        case 502:
          return "Bad Gateway";
        case 503:
          return "Service Unavailable";
        case 504:
          return "Gateway Timeout";
        case 505:
          return "HTTP Version Not Supported";
        case 506:
          return "Variant Also Negotiates";
        case 507:
          return "Insufficient Storage";
        case 508:
          return "Loop Detected";
        case 510:
          return "Not Extended";
        case 511:
          return "Network Authentication";
        default:
          break;
      }
    },
  },
};
</script>

<style>
#root {
  height: 92vh;
}

.no-selectable {
  -webkit-touch-callout: none; /* iPhone OS, Safari */
  -webkit-user-select: none; /* Chrome, Safari 3 */
  -khtml-user-select: none; /* Safari 2 */
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* IE10+ */
  user-select: none; /* Possível implementação no futuro */
}

.chip {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 4px;
  width: 86px;
  height: 38px;
  border-radius: 8px;
  background-color: #1f2937;
  color: #6b7280;
  border: 1px solid #6b7280;
}
.chip-status {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 4px;
  width: 86px;
  height: 38px;
  border-radius: 8px;
  border: 1px solid red;
}
.status500 {
  background-color: #e1bee7;
  color: #8e24aa;
  border: 1px solid #8e24aa;
}

.status400 {
  background-color: #f8bbd0;
  color: #c2185b;
  border: 1px solid #c2185b;
}

.status300 {
  background-color: #ffe0b2;
  color: #854d0e;
  border: 1px solid #854d0e;
}

.status200 {
  background-color: #b9f6ca;
  color: #166534;
  border: 1px solid #166534;
}
.status-default {
  background-color: #1f2937;
  color: #6b7280;
  border: 1px solid #6b7280;
}

.parent {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  grid-template-rows: repeat(6, 1fr);
  grid-column-gap: 8px;
  grid-row-gap: 8px;
  height: 100%;
}

.div1 {
  grid-area: 1 / 1 / 7 / 3;
}
.div2 {
  grid-area: 1 / 3 / 7 / 6;
}
.div3 {
  grid-area: 1 / 6 / 4 / 8;
}
.div4 {
  grid-area: 4 / 6 / 7 / 8;
}
</style>