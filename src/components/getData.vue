data: url = "../src/assets/questions.json"
axios
  .get("../src/assets/questions.json")
  .then(response => {
    console.log("response", response);
  })
  .catch(error => {
    console.log("error", error);
  });