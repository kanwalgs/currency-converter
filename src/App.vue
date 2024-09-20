<template>
  <div id="app">
    <h1>Currency Converter</h1>
    
    <!-- Display loader while processing -->
    <div v-if="showLoader">
      <img src='./assets/spinning-dots.svg' height='150px'/>
    </div>

    <!-- Main content -->
    <div v-else>
      <CurrencyInput @convert="convertCurrency" />

      <!-- Display converted result and image if available -->
      <div v-if="convertedDistribution && imageURL">
        <h2>Target value is {{ convertedDistribution }}</h2>
        <img :src="imageURL" alt="Generated Plot" />
      </div>
    </div>
  </div>
</template>

<script>
import CurrencyInput from './components/CurrencyInput.vue';
import axios from "axios";
import pako from "pako"; // For decompressing data

export default {
  data() {
    return {
      showLoader: false,
      convertedDistribution: null,
      imageURL: null,
      taskStdout: null,
      apiToken: process.env.VUE_APP_API_TOKEN,
      coreID: 'cor_0d65d1e8f57459079c0a036a3e09ae1d',
      retryCount: 0,
      retryDelay: 2000,
      maxRetryDelay: 30000,
    };
  },
  
  components: {
    CurrencyInput,
  },

  methods: {
    /**
     * Creates a task for currency conversion with uncertainty.
     */
    async createTask(applicationCode) {
      const taskRequest = {
        Type: "SourceCode",
        SourceCode: {
          Object: "SourceCode",
          Code: applicationCode,
          Arguments: "",
          Language: "C",
        },
        Overrides: {
          Core: this.coreID,
        },
      };

      try {
        const response = await fetch("https://api.signaloid.io/tasks", {
          method: "POST",
          headers: {
            "Authorization": this.apiToken,
            "Content-Type": "application/json",
          },
          body: JSON.stringify(taskRequest),
        });

        const result = await response.json();
        if (result.TaskID) {
          console.log(`Task created successfully with ID: ${result.TaskID}`);
          return result.TaskID;
        }
      } catch (error) {
        console.error("Error creating task:", error);
      }
    },

    /**
     * Polls the task status and retries until the task is completed.
     */
    async getTaskStatus(taskID) {
      try {
        const response = await fetch(`https://api.signaloid.io/tasks/${taskID}`, {
          method: "GET",
          headers: {
            "Authorization": this.apiToken,
            "Content-Type": "application/json",
          },
        });

        const result = await response.json();
        await this.handleTaskResponse(result, taskID);
      } catch (error) {
        console.error("Error getting task status:", error);
      }
    },

    /**
     * Handles task status updates, retries if necessary.
     */
    async handleTaskResponse(response, taskID) {
      const taskStatus = response.Status;
      console.log(`Task status: ${taskStatus}`);

      if (["Completed", "Cancelled", "Stopped"].includes(taskStatus)) {
        console.log(`Task in terminal state: ${taskStatus}. Fetching outputs.`);
        await this.getTaskOutput(taskID);
      } else {
        this.retryCount += 1;
        this.retryDelay = Math.min(2 ** this.retryCount * 1000, this.maxRetryDelay);
        console.log(`Task active: ${taskStatus}. Retrying in ${this.retryDelay}ms.`);
        setTimeout(() => this.getTaskStatus(taskID), this.retryDelay);
      }
    },

    /**
     * Retrieves task output and processes the first chunk.
     */
    async getTaskOutput(taskID) {
      try {
        const taskOutputsResponse = await axios.get(`https://api.signaloid.io/tasks/${taskID}/outputs`, {
          headers: { "Authorization": this.apiToken },
        });

        if (taskOutputsResponse.data.StdoutChunks.length) {
          const response = await axios.get(taskOutputsResponse.data.StdoutChunks[0], {
            responseType: "arraybuffer",
          });

          const decompressed = pako.inflate(new Uint8Array(response.data));
          const decompressedStdOutChunk = new TextDecoder("utf-8").decode(decompressed);

          const numericPart = decompressedStdOutChunk.split('Ux')[0];
          this.convertedDistribution = numericPart;

          // Extract Ux string and plot the distribution
          const uxString = this.extractUxString(decompressedStdOutChunk);
          if (uxString) {
            await this.plotDistribution(uxString);
          }
        }
      } catch (error) {
        console.error("Error fetching task output:", error);
      }
    },

    /**
     * Extracts Ux string from the task output.
     */
    extractUxString(taskOutput) {
      const match = taskOutput.match(/Ux[0-9a-fA-F]{40,}/);
      return match ? match[0] : null;
    },

    /**
     * Plots the distribution using the extracted Ux string.
     */
    async plotDistribution(uxString) {
      try {
        const response = await fetch(`https://api.signaloid.io/plot`, {
          method: "POST",
          headers: {
            "Authorization": this.apiToken,
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ payload: uxString }),
        });

        const result = await response.json();
        this.imageURL = result.presignedURL;
        this.showLoader = false;
      } catch (error) {
        console.error("Error plotting distribution:", error);
      }
    },

    /**
     * Main conversion logic, creates task and handles outputs.
     */
    async convertCurrency(amount, minRate, maxRate) {
      this.showLoader = true;

      const applicationCode = `
        #include <stdio.h>
        #include <stdlib.h>
        #include <time.h>

        double uniform(double min, double max) {
            return min + ((double) rand() / RAND_MAX) * (max - min);
        }

        int main() {
            srand(time(0));
            double conversionRate = uniform(${minRate}, ${maxRate});
            double convertedAmount = ${amount} * conversionRate;
            printf("Converted Amount: %f\\n", convertedAmount);
            return 0;
        }
      `;

      const taskID = await this.createTask(applicationCode);
      if (taskID) {
        console.log("Task created. Waiting for completion...");
        await this.getTaskStatus(taskID);
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
