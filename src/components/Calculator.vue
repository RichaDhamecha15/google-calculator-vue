<template>
  <div class="main">
    <div id="display" class="display">
      <div class="history">
        <div class="left history-icon">
          <svg focusable="false" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M13 3a9 9 0 0 0-9 9H1l3.89 3.89.07.14L9 12H6c0-3.87 3.13-7 7-7s7 3.13 7 7-3.13 7-7 7c-1.93 0-3.68-.79-4.94-2.06l-1.42 1.42A8.954 8.954 0 0 0 13 21a9 9 0 0 0 0-18zm-1 5v5l4.28 2.54.72-1.21-3.5-2.08V8H12z"></path></svg>
        </div>
        <div class="right">
          {{topDisplay}}
        </div>
      </div>
      <div class="output">{{bottomDisplay}}</div>
    </div>
    <div id="key" class="keys">
      <table class="left">
        <tbody>
          <tr v-for="(row, index) in calculatorKeys.scientificKeys" :key='index'>
            <td  v-for="(column, index) in row" :key='index' :colspan="column.colSpan">
              <div v-if="column.html" v-html="column.title"  :class="column.className" role="button" />
              <div v-else :class="column.className" role="button" >{{column.title}}</div>
            </td>
          </tr>
        </tbody>
      </table>
      <table class="right">
        <tbody>
          <tr v-for="(row, index) in calculatorKeys.digitalKeys" :key='index'>
            <td  v-for="(column, index) in row" :key='index' :colspan="column.colSpan">
              <div v-if="column.html" v-html="column.title" :class="column.className" role="button"  @click="onButtonClick(column)"/>
              <div v-else :class="column.className" role="button"  @click="onButtonClick(column)">{{column.title}}</div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { evaluate } from "mathjs";
export default {
  name: "Calculator",
  data() {
    return {
      supportedOperations: ['÷', '×', '+', '-'],
      currentInput: '',
      currentExpression: '',
      showExpression: false,
      historyExpressions: [],
      calculatorKeys:{
        scientificKeys:[
          [
            {title:'<div class="flex-item">Rad </div><div class="divider"></div><div class="flex-item disabled">Deg</div>', className:'operator rad-degree flex', colSpan: '2', html: true },
            {title:'x!', className:'operator'}
          ],
          [
            {title:'Inv', className:'operator'},
            {title:'sin', className:'operator'}, 
            {title:'ln', className:'operator'}
          ],
          [
            {title:'π', className:'operator'},
            {title:'cos', className:'operator'}, 
            {title:'log', className:'operator'}
          ],
          [
            {title:'e', className:'operator'},
            {title:'tan', className:'operator'}, 
            {title:'√', className:'operator'}
          ],
          [
            {title:'Ans', className:'operator'},
            {title:'EXP', className:'operator'}, 
            {title:'x<sup>y</sup>', className:'operator', html:true,}
          ]
        ],
        digitalKeys:[
          [
            {title:'(', className:'operator'},
            {title:')', className:'operator'}, 
            {title:'%', className:'operator'},
            {title:'AC', className:'operator'}
          ],
          [
            {title:'7', className:'digit'},
            {title:'8', className:'digit'}, 
            {title:'9', className:'digit'},
            {title:'÷', className:'operator symbol'}
          ],
          [
            {title:'4', className:'digit'},
            {title:'5', className:'digit'}, 
            {title:'6', className:'digit'},
            {title:'×', className:'operator symbol'}
          ],
          [
            {title:'1', className:'digit'},
            {title:'2', className:'digit'}, 
            {title:'3', className:'digit'},
            {title:'-', className:'operator symbol'}
          ],
          [
            {title:'0', className:'digit'},
            {title:'.', className:'digit symbol'}, 
            {title:'=', className:'equals'},
            {title:'+', className:'operator symbol'}
          ],
        ]
      }
    };
  },
  computed:{
      topDisplay(){
        let display;
        if(!this.historyExpressions.length && !this.currentExpression) return null;
        if(!this.showExpression){
          if(!this.historyExpressions.length && this.currentExpression) return `Ans = 0`;
          else display = 'Ans = ' + evaluate(this.historyExpressions[this.historyExpressions.length - 1].toString().replace(' ', '').replace(/×/g, '*').replace(/÷/g, '/'));
        }
        else display = this.historyExpressions[this.historyExpressions.length - 1] + ' =';
        return display;
      },
      bottomDisplay(){
        let display;
        if(!this.currentExpression) return 0;
        if(this.showExpression){
          const calculationString = this.currentExpression.toString().replace(' ', '').replace(/×/g, '*').replace(/÷/g, '/');
          display = evaluate(calculationString);
        }
        else display = this.currentExpression;
        return display;
      }
  },
  methods: {
    onButtonClick(key){
      if(key.className.includes('operator') && key.title === 'AC') {
          this.currentExpression = '';
          this.currentInput = '';
          this.showExpression = false;
          return;
      }
      if(key.className.includes('operator') && this.supportedOperations.includes(key.title)){        
        if(this.currentExpression || (!this.currentExpression && key.title === '-')) {
          this.currentExpression += ' ' + key.title + ' ';
          this.currentInput = '';          
          this.showExpression = false;
        }
      }
      if(key.className.includes('digit')){
        if((!this.currentInput || this.currentInput.includes('.')) && key.title === '.') return;
        if(this.showExpression){
          this.currentInput = '';
          this.currentExpression = '';
        }
        this.currentInput += key.title;
        this.currentExpression += key.title;
        this.showExpression = false;
      }

      if(key.title === '='){        
        this.calculateOutput();
      }
    },
    calculateOutput(){
      const calculationString = this.currentExpression.toString().replace(' ', '').replace(/×/g, '*').replace(/÷/g, '/');
      this.historyExpressions.push(this.currentExpression);
      this.currentExpression = evaluate(calculationString);
      this.showExpression = true;
    }
  }
};
</script>


<style lang="scss" scoped>
.main{
  width: 100%;
  max-width: 800px;
  height: auto;
  font-family: Roboto,Helvetica Neue,Arial,sans-serif;
  display: block;
  margin: 4px;
  &:hover{
    .display{
      border: 1px solid #dadce0; 
      box-shadow: inset 0 1px 1px rgba(0, 0, 0, 10%);
      border-top: 1px solid #70757a;
    }
  }
  
}

.display{
  margin: 4px;
  margin-bottom: 8px;
  border: 1px solid #ebebeb;  
  border-radius: 8px;
  height: 72px;
  text-align: right;
  padding: 10px 16px 0px 10px;
  box-sizing: border-box;
  line-height: 1;
  display: flex;
  flex-direction: column;
  .history{
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    .left{
      flex: 0.2;
      height: 22px;
      line-height: 22px;
      max-width: 22px;
      color: #70757a;
      fill: currentColor;
    }
    .right{
      flex: 0.8;
      font-size: 13px;
      color: #70757B;   
    }
  }
  .output{
    font-size: 30px;
    height: 32px;
    vertical-align: bottom;
    color: #4d5156;
  }
  &.active{
    border: 1px solid #4285f4; 
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, 30%);
  }
}
.keys{
  margin: auto;
  padding: 0;
  display: flex;
  table{
    border-collapse: collapse;
    table-layout: fixed;    
    &.left{
      width: 42%;
    }
    &.right{
      width: 58%;
    }
  }
}
td {
  div{
    border: 1px solid #dadce0;
    color: #202124;
    display: block;
    cursor: pointer;
    font-family: Roboto,Helvetica Neue,Arial,sans-serif;
    font-size: 14px;
    text-align: center;
    line-height: 34px;
    margin: 4px;
    box-sizing: border-box;
    position: relative;
    border-radius: 4px;    
    &.flex{
      display: flex;
    }
    &.symbol{
      font-size: 20px
    }
  }
}
.operator{
  background-color: #dadce0;
  &:active{    
    background-color: #dadce0;
    box-shadow: 0 0 0 1px #70757a;
  }
}
.digit{
  background-color: #f1f3f4;
  font-size: 15px;
  &:active{
    background-color: #f1f3f4; 
    box-shadow: 0 0 0 1px #dadce0;
  }
}
.equals{
  color: #fff;
  background-color: #4285f4;
  border: 1px solid #4285f4;
  font-size: 20px;
  font-weight: bold;
  &:active{    
    background-color: #4285f4;
    box-shadow: 0 1px 2px 0 rgba(66,133,244, 45%), 0 3px 6px 2px rgba(66, 133, 244, 30%);
  }
}
</style>
<style lang="scss">
.rad-degree{  
  .flex-item{
    flex: 1;
  }
  .disabled{
    color: #70757a
  }
  .divider{
    content: "";
    position: absolute;
    top: 10px;
    bottom: 10px;
    left: 50%;
    border-left: 1px solid #70757a;
    transform: translateX(-50%);
  }
}
</style>