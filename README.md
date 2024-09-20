# Currency Converter
This is a single-page Vue.js application that converts a given amount between currencies, taking into account an uncertainty range for the conversion rate. The application leverages the Signaloid API to generate a distribution of converted values and visualizes the output.

## Features
- Input an amount and specify a range for conversion rates.

- Process and visualize the conversion results using the Signaloid API.

- Dynamically fetches and displays a distribution plot image.

## Installation and Running the Application
### Prerequisites
Node.js and npm should be installed on your machine. If you don’t have them installed, follow this guide to install them.
### Clone the Repository
```
git clone https://github.com/kanwalgs/currency-converter.git

cd currency-converter
```
### Install Dependencies
```
npm install
```
### Run the Development Server
```
npm run serve
```
This will start the Vue.js development server at http://localhost:8080.

### Build for Production
To create a production build, run:
```
npm run build
```
### Authentication Token
The application interacts with the Signaloid API using an API token. For security reasons, it is not hardcoded into the source file, but rather in an .env file.

#### How to add the token
Create a .env file at the root of the project:
```
VUE_APP_API_TOKEN=your_signaloid_api_token
```
It will be accessed in the code like this (you don't need to change the code):
```
const apiToken = process.env.VUE_APP_API_TOKEN;
```
Git Ignore: Ensure your .env file is added to .gitignore so that sensitive data isn't pushed to your repository.

### Running with an Environment File
After adding your API token in a .env file, run:
```
npm run serve
```
## Walkthrough of the Code
### Components
CurrencyInput.vue: This component allows the user to input the amount and range (minRate, maxRate) for the conversion.

### Main App:
- It listens for the user's input and calls the convertCurrency function to initiate the API task.

- The app sends source code (C code) to the Signaloid Cloud Compute Engine, which calculates a random conversion rate within the specified range.

- Once the task is completed, it retrieves the output, processes the distribution of converted values, and fetches a presigned URL for the plot visualization.

### Main Methods
- **createTask:** This method sends the C code to the API to initiate a task for calculating the conversion with uncertainty.

- **getTaskStatus:** Polls the API for the task status and fetches the task output when the task is completed.

- **getTaskOutputsChunks:** Decompresses the output data (which includes the conversion results) and extracts the distribution value.

- **plotDistribution:** Sends the Ux string to the API to generate a distribution plot image and updates the application with the presigned URL for display.

### Loader
The application shows a loader (spinning-dots) while the conversion task is being processed asynchronously.

###
