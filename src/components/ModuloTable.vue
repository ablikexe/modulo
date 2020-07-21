<template>
  <div>
    <div>
      Current modulo: <b>{{ modulo }}</b>.<br>
      Factorization: {{ factorize(modulo).join('*') }}.<br>
      &phi;(n) = {{ phi(modulo) }}.<br>
      Browse all modulos:
      <button :disabled="modulo <= 1" @click="changeModulo(-1)">-1</button>
      <button @click="changeModulo(1)">+1</button><br>
      Browse interesting modulos:
      <button :disabled="modulo <= 6" @click="findInterestingModulo(-1)">prev</button>
      <button @click="findInterestingModulo(1)">next</button>
    </div>
    <div class="table-wrapper">
      <table>
        <th></th>
        <!-- Keys workaround from https://github.com/vuejs/vue/issues/7323 -->
        <th v-for="col in cols" :key="'header' + col">{{ col }}</th>
        <tr v-for="row in rows" :key="'row' + row" v-bind:class="highlightRow(row)">
          <th>{{ row }}</th>
          <td
            v-bind:class="highlight(row, col)"
            v-for="col in cols"
            :key="'row' + row + '-col' + col"
          >{{getValue(row, col)}}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
const INITIAL_MODULO = 35, MIN_COLS = 30, MIN_ROWS = 30, ELLIPSIS = '...';
export default {
  name: "ModuloTable",
  methods: {
    getValue(x, y) {
      if (y === ELLIPSIS) return '...';

      // Calculate x ** y manually, with a loop.
      // This way we can take the result modulo this.modulo after each multiplication
      // which avoids using big numbers and getting wrong answers.
      // We could do the same with the faster algorithm from the previous exercise.
      let res = 1;
      for (let i=0; i<y; i++) res = (res * x) % this.modulo;
      return res;
    },
    highlight(x, y) {
      return {
        active: this.getValue(x, y) === 1,
        zero: this.getValue(x, y) === 0,
        crucial: y%this.phi(this.modulo) === 1,
      };
    },
    highlightRow(x) {
      // Checks if x and this.modulo are coprime (they have no common divisors).
      // It can be done much faster using Euclidean algorithm.
      for (let i=2; i<=Math.min(x, this.modulo); i++)
        if (x%i === 0 && this.modulo%i === 0)
          return { notcoprime: true };
      return { coprime: true };
    },
    getArray(length) {
      const arr = [];
      for (let i = 1; i <= length; i++) {
        arr.push(i);
      }
      return arr;
    },
    refreshTable() {
      this.rows = this.getRows(this.modulo);
      this.cols = this.getCols(this.modulo);
    },
    changeModulo(diff) {
      this.modulo += diff;
      this.refreshTable();
    },
    twoDifferentValues(arr) {
      return arr.length === 2 && arr[0] !== arr[1];
    },
    findInterestingModulo(diff) {
      do {
        this.modulo += diff;
      } while (!this.twoDifferentValues(this.factorize(this.modulo)));
      this.refreshTable();
    },
    factorize(n) {
      let primeFactors = [];
      for (let i=2; i<=n; i++) {
        while (n%i === 0) {
          primeFactors.push(i);
          n /= i;
        }
      }
      return primeFactors;
    },
    phi(n) {
      let factors = this.factorize(n);
      let last = null;
      let res = 1;
      for (let x of factors) {
        res *= (x === last ? x : x-1);
        last = x;
      }
      return res;
    },
    getRows(n) {
      return this.getArray(Math.max(n, MIN_ROWS));
    },
    getCols(n) {
      let p = this.phi(n);
      if (p <= MIN_COLS+1) return this.getArray(Math.max(p+1, MIN_COLS));
      else return [...this.getArray(MIN_COLS), ELLIPSIS, p, p+1];
    }
  },
  data: function() {
    return {
      modulo: INITIAL_MODULO,
      rows: this.getRows(INITIAL_MODULO),
      cols: this.getCols(INITIAL_MODULO),
    };
  }
};
</script>

<style scoped>
.active {
  background-color: yellow;
}
.zero {
  background-color: red;
}
.crucial {
  background-color: #af6;
}
.coprime {
  opacity: 0.8;
}
.notcoprime {
  opacity: 0.4  ;
}
.table-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 3em;
}
td {
  padding: 1em;
  width: 1em;
  height: 1em;
  border: 1px solid silver;
}
</style>
