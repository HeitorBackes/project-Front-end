<template>
  <div class="container my-5 p-0">
    <div class="row gap-5">
      <div class="inputs col">
        <h1 class="fw-bold display-1">Orçamentos para hotéis</h1>
        <p>Preencha os campos abaixo para gerar um orçamento.</p>
        <div class="input-wrapper row">
          <div class="var-wrapper col-12 col-md-6">
            <label for="dataCheckIn" class="form-label">Data do Check-In</label>
            <input type="date" class="form-control" v-model="dataCheckIn" />
          </div>

          <div class="var-wrapper col-12 col-md-6">
            <label for="dataCheckOut" class="form-label"
              >Data do Check-Out</label
            >
            <input type="date" class="form-control" v-model="dataCheckOut" />
          </div>

          <div class="var-wrapper col-12 col-md-6">
            <label for="numGuests" class="form-label">Número de pessoas</label>
            <input
              type="number"
              class="form-control"
              v-model="numGuests"
              placeholder="Ex: 2"
            />
          </div>

          <div class="var-wrapper col-12 col-md-6">
            <label for="discountRate" class="form-label">Desconto (%)</label>
            <input type="number" class="form-control" v-model="discountRate" />
          </div>
        </div>
        <div class="row my-3">
          <div class="col">
            <button
              @click="generateBudget"
              class="btn btn-primary px-3 py-2 w-100"
            >
              Gerar orçamento
            </button>
          </div>
          <div class="col">
            <button
              class="btn btn-outline-primary px-3 py-2 w-100"
              @click="copyBudget"
            >
              Copiar orçamento
            </button>
          </div>
        </div>
        <div class="output">
          <p v-html="outputMessage"></p>
        </div>
      </div>

      <div class="outputs col my-2">
        <div
          id="div-output"
          class="bg-light border border-secondary rounded-3 p-2 mb-2 border-opacity-25 p-4 h-100"
        >
          <p id="output" v-html="output"></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      values: {
        single: 140,
        twin: 160,
        double: 180,
        triple: 225,
        quad: 270,
      },
      dataCheckIn: null,
      dataCheckOut: null,
      numGuests: null,
      numDaily: null,
      discountRate: 0,
      output: "",
      outputMessage: "",
    };
  },

  methods: {
    formatDate(isoDate) {
      if (!isoDate) return "";
      const [year, month, day] = isoDate.split("-");
      return `${day}/${month}/${year}`;
    },

    copyBudget() {
      if (!this.output) {
        this.outputMessage = "Nenhum orçamento disponível para copiar.";
        return;
      }

      let formattedOutput = this.output
        .replace(/<br>/g, "\n")
        .replace(/(\n\s*)+/g, "\n")
        .trim();

      navigator.clipboard
        .writeText(formattedOutput)
        .then(() => {
          this.outputMessage =
            "Orçamento copiado para a área de transferência!";
        })
        .catch((error) => {
          console.error("Erro ao acessar o clipboard:", error);
          this.outputMessage = "Erro ao copiar o orçamento.";
        });
    },

    calcNumDaily() {
      console.log("Data Check-In:", this.dataCheckIn);
      console.log("Data Check-Out:", this.dataCheckOut);

      if (this.dataCheckIn && this.dataCheckOut) {
        const checkIn = new Date(this.dataCheckIn);
        const checkOut = new Date(this.dataCheckOut);
        console.log("Check-In Date Object:", checkIn);
        console.log("Check-Out Date Object:", checkOut);

        const timeDifference = checkOut - checkIn;
        console.log("Time Difference (ms):", timeDifference);

        if (timeDifference > 0) {
          this.numDaily = timeDifference / (1000 * 60 * 60 * 24);
          console.log("Number of Days:", this.numDaily);
          return true;
        } else {
          this.numDaily = 0;
          this.output = "A data de Check-Out deve ser posterior ao Check-In.";
          return false;
        }
      }

      this.numDaily = 0;
      this.output = "Por favor, selecione as datas.";
      return false;
    },

    calcDiscount(price, discount) {
      return price - price * (discount / 100);
    },
    getPrice() {
      const { single, twin, double, triple, quad } = this.values;
      const { numGuests, numDaily, discountRate } = this;

      let result = "";

      if (numGuests <= 1) {
        result += `*Quarto Individual*<br>
Valor por diária: R$ ${Math.round(this.calcDiscount(single, discountRate))}<br>
Valor total: R$ ${
          Math.round(this.calcDiscount(single, discountRate)) * numDaily
        }<br><br>\n`;
      }
      if (numGuests <= 2) {
        result += `*Quarto Duplo*<br>
Valor por diária: R$ ${Math.round(this.calcDiscount(twin, discountRate))}<br>
Valor total: R$ ${
          Math.round(this.calcDiscount(twin, discountRate)) * numDaily
        }<br><br>\n
*Quarto Casal*<br>
Valor por diária: R$ ${Math.round(this.calcDiscount(double, discountRate))}<br>
Valor total: R$ ${
          Math.round(this.calcDiscount(twin, discountRate)) * numDaily
        }<br><br>\n`;
      }
      if (numGuests <= 3) {
        result += `*Quarto Triplo*<br>
Valor por diária: R$ ${Math.round(this.calcDiscount(triple, discountRate))}<br>
Valor total: R$ ${
          Math.round(this.calcDiscount(triple, discountRate)) * numDaily
        }<br><br>\n`;
      }
      if (numGuests <= 4) {
        result += `*Quarto Quadruplo*<br>
Valor por diária: R$ ${Math.round(this.calcDiscount(quad, discountRate))}<br>
Valor total: R$ ${
          Math.round(this.calcDiscount(quad, discountRate)) * numDaily
        }<br><br>`;
      }

      if (!result) {
        result = "Não temos quartos disponíveis para este número de pessoas.";
      }

      return result;
    },
    generateBudget() {
      this.calcNumDaily();

      if (!this.numDaily || !this.numGuests) {
        this.outputMessage =
          "Selecione uma data válida ou o número de pessoas.";
        return;
      }

      this.output = `
Data de Check-In: ${this.formatDate(this.dataCheckIn)}<br>
Data de Check-Out: ${this.formatDate(this.dataCheckOut)}<br>
Número de Diárias: ${this.numDaily}<br>
Número de Pessoas: ${this.numGuests}<br>
Desconto: ${this.discountRate}%<br><br>
${this.getPrice()}`;
      this.outputMessage = "Seu orçamento foi gerado com sucesso!";
    },
  },
};
</script>

<style scoped>
</style>
