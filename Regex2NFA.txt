
function Infix2Postfix(infix)  //the function to convert the regex infix to postfix version
{
  var stack = new Array();
  var queue = new Array();
  
  var operators={
    "*":{
         precedence:4,
         associativity:"Left"
        },
    ".":
        {
          precedence:3,
          associativity:"Left"
        },
    "+":
        {
          precedence:2,
          associativity:"Left"
        }
  }
  
  for(var i=0;i<infix.length;i++)
  {
    var token = infix[i];
    
    if(token.match(/[a-z]/))
    {
      queue.push(token);
    }
    else if("*.+".indexOf(token)!==-1)
    {
      var op1 = token;
      var op2 = stack[stack.length-1];
      
      while("*.+".indexOf(op2)!==-1 && (operators[op1].associativity=="Left" && operators[op1].precedence<=operators[op2].precedence))
      {
       queue.push(stack.pop());
       op2=stack[stack.length-1]; 
      }
      
      stack.push(op1);
    }
    else if(token=="(")
      {
        stack.push(token);
      }
    else if(token==")")
      {
        while(stack[stack.length-1]!=="(")
          {
            queue.push(stack.pop());
          }
        stack.pop();
      }
  }
  while(stack.length>0)
    {
      queue.push(stack.pop());
    }
  
  return queue;
}

hashNFA = {};  //it represents our nfa machine
acceptArray = []; //it keeps the accept states
checkedArray=[]; /*it keeps the states which is controlled already. 
                   this array not allows to check a state more than one*/
var k = 0;
function checkAccept(state){  //the function of selecting all the accept states
    if(!(checkedArray.includes(state.label))){  
        checkedArray.push(state.label);
        if(state.acceptState!=false){    
            acceptArray[k] = state.label;
            k++;
        }
        if(state.nextState != '' ){  //recursive call for the state's next state
            checkAccept(hashNFA[state.nextState]);
        }
        if(state.epsilon != undefined){  //recursive call for the state's epsilon array
            for(var j=0;j<state.epsilon.length;j++){
                checkAccept(hashNFA[state.epsilon[j]]);
            }
        }
    }
}

function NFA(queue) //function converts the regex to nfa
{
  var stackNFA = new Array();
  var tempQueue; //not to lose the queue value
  var tempNFA; //used for to apply the operator to the operand
  var tempNFA2; //used for to apply the operator '+' and '.'
  var node=0;  //holds the state number
  var length = queue.length;
  for(var i=0;i<length-1;i++)
    {
      tempQueue=queue.shift(); 
      if(tempQueue.match(/[a-z]/))  //creting a new state machine of a transition
        {
            
          hashNFA[node]={
                          label:node,
                          nextState:node+1,
                          epsilon:[],
                          transition:tempQueue,
                          acceptState:false
                        }
          
          hashNFA[node+1]={
                            label:node+1,
                            nextState:'',
                            epsilon:[],
                            transition:'',
                            acceptState:true
                          }
          
          stackNFA.push(hashNFA[node]);
          node+=2;
        }
      else if("*.+".indexOf(tempQueue)!==-1) // applying the operator rules
        {
          if("*".indexOf(tempQueue)!==-1)
            {

              tempNFA=stackNFA.pop();
              checkAccept(tempNFA); 
              k=0;
              checkedArray=[];
              
              for(var i=0;i<acceptArray.length;i++){ //link accept states to the old initial state
                  hashNFA[acceptArray[i]].epsilon.push(tempNFA.label);
              }
              
              hashNFA[node]={ //adding new state
                            label:node,
                            nextState:'',
                            epsilon:[tempNFA.label],
                            transition:'',
                            acceptState:true
                            }
                
                tempNFA = hashNFA[node]; //new initial state

                node++;
                stackNFA.push(tempNFA);
                
                
            
            }
            else if(".".indexOf(tempQueue)!==-1)
            {
              tempNFA2 = stackNFA.pop();
              tempNFA = stackNFA.pop();
              
              checkAccept(tempNFA);
              
              k=0;
              checkedArray=[];
              
              for(var j=0;j<acceptArray.length;j++){  /*link the acceptStates to the last accept state
                                                        and remove their acceptness*/
                  hashNFA[acceptArray[j]].epsilon.push(tempNFA2.label);
                  hashNFA[acceptArray[j]].acceptState=false;
              }
              checkAccept(tempNFA);
              
                      
              stackNFA.push(tempNFA);
              
            }
            else if("+".indexOf(tempQueue)!==-1)
            {
              tempNFA2 = stackNFA.pop();
              tempNFA = stackNFA.pop();
              
              hashNFA[node]={  //adding new state 
                            label:node,
                            nextState:'',
                            epsilon:[tempNFA.label,tempNFA2.label],
                            transition:'',
                            acceptState:false
                            }
                            
                stackNFA.push(hashNFA[node]);
                node++;
            }
             
          acceptArray=[];
        }
    }
    return stackNFA.pop();
}

regex=prompt("Enter the regular expression","");
str = prompt("Enter the string","");

nonDetAuto=NFA(Infix2Postfix(regex));

