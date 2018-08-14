<template>
  <div class="calculator center">
    <div ref="display" class="display align displayfontsize0"> {{ display || 0 }} </div>
    <div ref="clear" @mousedown="highlight" @mouseout="nohighlight" @mouseup="clear" class="btn align topoperator clear">AC</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="sign" class="btn align topoperator">⁺∕₋</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="percent" class="btn align topoperator percent">%</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="division" class="btn align sideoperator division">÷</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">7</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">8</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">9</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="multiplication" class="btn align sideoperator multiplication">×</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">4</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">5</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">6</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="subtraction" class="btn align sideoperator subtraction">−</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">1</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">2</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">3</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="addition" class="btn align sideoperator">+</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">0</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num"></div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="digit" class="btn align num">.</div>
    <div @mousedown="highlight" @mouseout="nohighlight" @mouseup="equals" class="btn align sideoperator">=</div>
  </div>
</template>

<script>
import BigNumber from '../assets/bignumber';

BigNumber.config({
  RANGE: [-99, 160],
  EXPONENTIAL_AT: [-16, 16],
  DECIMAL_PLACES: 99,
});
const PRECISION = 16;

const [DIGIT, ENTER, ADDITION, SUBTRACTION, MULTIPLICATION, DIVISION] = [
  -1,
  0,
  1,
  2,
  3,
  4,
  5,
];

let display; // display font resizing
let clear; // clear button text value updates
let blockmouseup = false; // block key operation if mouse was dragged across while pressed

const template = {
  leftoperand: '',
  rightoperand: '',
  previous: '',
  current: DIGIT,
  operator: null,
  display: '0',
};

const adjustDisplay = function adjustDisplay() {
  const FONTBREAKPOINT = [7, 9, 11, 13, 15];
  const len = this.display.length;

  let i = FONTBREAKPOINT.slice()
    .reverse()
    .findIndex(e => len > e);

  i = i === -1 ? 0 : 5 - i;

  for (let j = 0; j <= FONTBREAKPOINT.length; j += 1) {
    if (i === j) display.classList.add(`displayfontsize${j}`);
    else display.classList.remove(`displayfontsize${j}`);
  }
};

const operationsPrep = function operations() {
  if (this.current === ENTER) {
    this.leftoperand = this.display.slice(0);
  } else if (this.previous === ENTER) {
    this.leftoperand = this.display.slice(0);
  } else {
    this.rightoperand = this.display.slice(0);
  }
};

const operatorClick = function operatorClick() {
  if (this.operator === null) {
    this.leftoperand = this.display.slice(0);
  } else {
    this.rightoperand = this.display.slice(0);
    if (this.current === DIGIT) {
      this.display = this.operator()
        .sd(PRECISION)
        .toString();
      this.leftoperand = this.display.slice(0);
      adjustDisplay.call(this);
    }
  }
};

export default {
  name: 'Calculator',
  mounted() {
    ({ display } = this.$refs);
    ({ clear } = this.$refs);
  },
  methods: {
    highlight(e) {
      blockmouseup = false;
      e.target.classList.add('highlight');
      if (e.target.textContent.localeCompare('0') === 0) {
        e.target.nextSibling.classList.add('highlight');
      }
      if (e.target.textContent.localeCompare('') === 0) {
        e.target.previousSibling.classList.add('highlight');
      }
    },
    nohighlight(e) {
      blockmouseup = true;
      e.target.classList.remove('highlight');
      if (e.target.textContent.localeCompare('0') === 0) {
        e.target.nextSibling.classList.remove('highlight');
      }
      if (e.target.textContent.localeCompare('') === 0) {
        e.target.previousSibling.classList.remove('highlight');
      }
    },
    clear(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      this.display = '0';
      if (clear.textContent.localeCompare('C') === 0) {
        clear.textContent = 'AC';
      } else {
        Object.assign(this, template);
      }
    },
    sign(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      this.display = (BigNumber(this.display) * -1).toString();
      adjustDisplay.call(this);
    },
    percent(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      this.display = BigNumber(this.display)
        .dividedBy(BigNumber(100))
        .sd(PRECISION)
        .toString();
      adjustDisplay.call(this);
    },
    digit(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      if (e.target.textContent.localeCompare('0') === 0) {
        e.target.nextSibling.classList.remove('highlight');
      }
      if (e.target.textContent.localeCompare('') === 0) {
        e.target.previousSibling.classList.remove('highlight');
      }
      let { textContent } = e.target;
      if (textContent.localeCompare('') === 0) textContent = '0'; // button to right of zero
      if (this.current !== DIGIT) {
        // no append, because last key was an operator or enter
        this.display = textContent;
        if (textContent.localeCompare('.') === 0) {
          this.display = '0.'; // prepend dots with 0
        }
      } else if (this.display.localeCompare('0') === 0) {
        // no append, because there is a zero on the screen
        this.display = textContent;
        if (textContent.localeCompare('.') === 0) {
          this.display = '0.'; // prepend dots with 0
        }
      } else if (
        textContent.localeCompare('.')
        || this.display.indexOf('.') === -1
      ) {
        // append allowed in other cases, but do not allow double dot
        this.display += textContent;
      }
      if (this.current !== DIGIT) {
        // ignore repeated digits to avoid losing track of operators
        this.previous = this.current;
        this.current = DIGIT;
      }
      clear.textContent = 'C';
      adjustDisplay.call(this);
    },
    addition(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      if (this.current === DIGIT || this.current === ENTER) {
        operatorClick.call(this, e);
      }

      this.operator = function operator() {
        return BigNumber(this.leftoperand).plus(BigNumber(this.rightoperand));
      };
      this.previous = this.current;
      this.current = ADDITION;
    },
    subtraction(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      if (this.current === DIGIT || this.current === ENTER) {
        operatorClick.call(this, e);
      }
      this.operator = function operator() {
        return BigNumber(this.leftoperand).minus(BigNumber(this.rightoperand));
      };
      this.previous = this.current;
      this.current = SUBTRACTION;
    },
    division(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      if (this.current === DIGIT || this.current === ENTER) {
        operatorClick.call(this, e);
      }
      this.operator = function operator() {
        return BigNumber(this.leftoperand).dividedBy(
          BigNumber(this.rightoperand),
        );
      };
      this.previous = this.current;
      this.current = DIVISION;
    },
    multiplication(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      if (this.current === DIGIT || this.current === ENTER) {
        operatorClick.call(this, e);
      }
      this.operator = function operator() {
        return BigNumber(this.leftoperand).multipliedBy(
          BigNumber(this.rightoperand),
        );
      };
      this.previous = this.current;
      this.current = MULTIPLICATION;
    },
    equals(e) {
      if (blockmouseup) return;
      e.target.classList.remove('highlight');
      if (this.operator) {
        operationsPrep.call(this);
        this.display = this.operator()
          .sd(PRECISION)
          .toString();
        this.leftoperand = this.display.slice(0);
        adjustDisplay.call(this);
      }
      this.previous = this.current;
      this.current = ENTER;
    },
  },
  data() {
    return Object.assign({}, template);
  },
};
</script>

<style scoped>
:root {
  --calc-width: 232px;
  --calc-height: 320px;
  --outer-border-top: 1px solid rgb(153, 153, 153);
  --outer-border: 1px solid rgb(104, 104, 104);
  --inner-border: 1px solid rgb(142, 142, 142);
  --num-background-color: rgb(224, 224, 224);
  --num-background-selected-color: rgb(178, 178, 178);
  --side-operator-background-color: rgb(243, 147, 61);
  --side-operator-background-selected-color: rgb(196, 116, 47);
  --top-operator-background-color: rgb(214, 214, 214);
  --top-operator-background-selected-color: rgb(169, 169, 169);
  --display-background-color: rgb(153, 153, 153);
  --button-font-color: black;
  --button-font-size: 23px;
  --button-font-weight: 400;
  --display-font-color: white;
  --side-operator-font-color: white;
  --side-operator-selected-font-color: rgb(85, 85, 85);
  --display-font-size0: 49px;
  --display-font-size1: 39px;
  --display-font-size2: 30px;
  --display-font-size3: 26px;
  --display-font-size4: 22px;
  --display-font-size5: 16px;
  --display-font-weight: 200;
  --button-clear-font-size: 18px;
  --button-percent-font-size: 17px;
  --button-sideoperator-font-size: 28px;
}

.calculator {
  font-family: "Helvetica Neue", "Roboto", sans-serif;
  -webkit-font-smoothing: antialiased;
  width: var(--calc-width);
  height: var(--calc-height);
  overflow: hidden;
  user-select: none;
  border-top: var(--outer-border-top);
  border-left: var(--outer-border);
  border-right: var(--outer-border);
  border-bottom: var(--outer-border);
  border-radius: 5px;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: 80px repeat(5, 1fr);
}

.display {
  white-space: nowrap;
  color: var(--display-font-color);
  font-weight: var(--display-font-weight);
  background-color: var(--display-background-color);
  grid-column: 1 / span 4;
}

.displayfontsize0 {
  font-size: var(--display-font-size0);
}
.displayfontsize1 {
  font-size: var(--display-font-size1);
  padding-bottom: 2px;
}
.displayfontsize2 {
  font-size: var(--display-font-size2);
  padding-bottom: 4px;
}
.displayfontsize3 {
  font-size: var(--display-font-size3);
  padding-bottom: 4px;
}
.displayfontsize4 {
  font-size: var(--display-font-size4);
  padding-bottom: 6px;
}
.displayfontsize5 {
  font-size: var(--display-font-size5);
  padding-bottom: 8px;
}

.btn {
  color: var(--button-font-color);
  font-size: var(--button-font-size);
  font-weight: var(--button-font-weight);
}

.calculator.center {
  position: fixed;
  top: 50%;
  left: 50%;
  margin-left: calc(var(--calc-width) / -2);
  margin-top: calc(var(--calc-height) / -2);
}

.display.align {
  display: grid;
  align-items: end;
  text-align: right;
  padding-right: 15px;
}

.btn.align {
  display: grid;
  align-items: center;
  text-align: center;
}

.btn:nth-child(4n + 2):nth-child(-n + 14),
.btn:nth-child(4n + 3):nth-child(-n + 15),
.btn:nth-child(4n + 4):nth-child(-n + 16) {
  border-right: var(--inner-border);
  border-bottom: var(--inner-border);
}

.btn:nth-child(4n + 5):nth-child(-n + 17) {
  border-bottom: var(--inner-border);
}

.btn:nth-child(19),
.btn:nth-child(20) {
  border-right: var(--inner-border);
}

.num {
  background-color: var(--num-background-color);
}

.sideoperator {
  padding-bottom: 5px;
  color: var(--side-operator-font-color);
  background-color: var(--side-operator-background-color);
  font-size: var(--button-sideoperator-font-size);
}

.topoperator {
  background-color: var(--top-operator-background-color);
}

.clear {
  font-size: var(--button-clear-font-size);
}

.percent {
  font-size: var(--button-percent-font-size);
}

.num.highlight {
  background-color: var(--num-background-selected-color);
}

.topoperator.highlight {
  background-color: var(--top-operator-background-selected-color);
}

.sideoperator.highlight {
  color: var(--side-operator-selected-font-color);
  background-color: var(--side-operator-background-selected-color);
}
</style>
