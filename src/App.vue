<template>
  <div class="ss-page">
    <pre>{{ mainMemoryVariables }}</pre>
    <!-- loads info -->
    <!-- <pre>{{ loadsInfo }}</pre> -->
    <!-- Instructions -->
    <!-- <pre>{{ instructions }}</pre> -->
    <label>
      <input v-model="processType" type="radio" value="FCFS" />
      FCFS
    </label>
    <label>
      <input v-model="processType" type="radio" value="SJF" />
      SJF
    </label>
    <label>
      <input v-model="processType" type="radio" value="SPN" />
      SPN
    </label>
    <label>
      <input v-model="processType" type="radio" value="SRTN" />
      SRTN
    </label>
    <label>
      <input v-model="processType" type="radio" value="NO_EXPROPIATIVE" />
      Prioridad no expropiativo
    </label>
    <label>
      <input v-model="processType" type="radio" value="EXPROPIATIVE" />
      Prioridad expropiativo
    </label>
    <p>
      Variables <br />
      <!-- Texto preformateado con saltos de linea -->
      
    </p>
    <p>
      Labels <br />
      <!-- Texto preformateado con saltos de linea -->
      <pre>{{ mainMemoryLabels }}</pre>
    </p>
    <!-- Barra de menu -->
    <div class="ss-menu">
      <span class="ss-file-submenu">
        <button>Archivo</button>
        <span class="ss-file-submenu__actions">
          <!-- accion del usuario al ingresar datos en el input -->
          <button>
            <label for="files" class="ss-open-action">Abrir</label>
          </button>
          <input
            id="files"
            style="visibility: hidden"
            type="file"
            @input="onLoadFile"
          />
        </span>
      </span>
      <button @click="onExecute">Ejecute</button>
      <button>Muestre Memoria</button>
      <button>Pausa</button>
      <button @click="isStepByStep = !isStepByStep">Paso a paso</button>
      <button>Salir</button>
    </div>
    
    <!-- Encabezado donde va el titulo y el acumulador -->
    <div class="ss-header">
      <h1>CH Máquina</h1>
      <div class="ss-accumulator">
        <!--{{}} Referencia reactiva: ref, proxy reflectivo, es una referencia en memoria de un valor global, no hay necesidad de refrescar la pagina  -->
        <h2>Acumulador {{ mainMemory[0].value }}</h2>
      </div>
    </div>
    <!-- Tabla de variables  -->
    <div class="ss-tables">
      <div class="ss-table__container ss-variables-table">
        <h2>Variables</h2>
        <table>
          <!-- {} para imprimir el valor en cada iteración. Para iterar se utiliza v-for. 
          para mostrar la propiedad del objeto, podemos agregar como segundo argumento una key que imprima el nombre de cada propiedad-->
          <tr
            v-for="(variable, variableIndex) in variableNames"
            :key="variableIndex"
          >
          <!-- hacemos uso de la funcion formatIndex para colocar la cantidad de 0 a cada instruccion -->
            <td>{{ formatIndex(variable.index) }} {{ variable.name }}</td>
          </tr>
        </table>
      </div>
      <!-- se añade la imagen de la computadra -->
      <div class="ss-laptop">
        <!-- se crea una funcion para verificar si esta en paso a paso -->
        <p>{{ isStepByStep ? "paso a paso" : "" }}</p>
        <span class="ss-laptop__value">{{ laptopValue }}</span>
        <img src="computadora.png" />
        
        <!-- se crea una funcion para verificar si esta en modo usuario o modo kernel -->
        <p>{{ isStepByStep ? "modo usuario" : "modo kernel" }}</p>

        <div class="ss-laptop">
          <span class="ss-laptop__value">{{ printerValue }}</span>
          <img src="printer.png" />
        </div>

        <div class="ss-laptop">
          <span class="ss-laptop__value">{{ emailValue }}</span>
          <img src="email.png" />
        </div>
      </div>
      <!-- Tabla de etiquetas  -->
      <div class="ss-table__container ss-labels-table">
        <h2>Etiquetas</h2>
        <table>
          <tr v-for="(label, labelIndex) in labels" :key="labelIndex">
            <td>{{ label.index }} {{ label.name }}</td>
          </tr>
        </table>
      </div>
      <!-- tabla donde carga el archivo CH -->
      <div class="ss-table__container ss-file-table">
        <h2>Archivo CH</h2>
        <table>
          <tr
            v-for="(instruction, instructionIndex) in instructions"
            :key="instructionIndex"
          >
            <td>
              {{ formatInstructionIndex(instructionIndex) }} {{ instruction }}
            </td>
          </tr>
        </table>
      </div>
      <div></div>
      <div class="ss-table__container ss-memory-table">
        <h2>Memoria Principal</h2>
        <table>
          <tr
            v-for="(memoryItem, memoryIndex) in mainMemory"
            :key="memoryIndex"
          >
          <!-- hacemos uso de la funcion formatIndex para colocar la cantidad de 0 a cada instruccion -->
            <td>{{ formatIndex(memoryIndex) }} {{ memoryItem.value }}</td>
          </tr>
        </table>
      </div>
    </div>
    <!-- is open es una propiedad de entrada definida del componenete modal, modal para ingresar el valor de la memoria y el kernel  -->
    <Modal :is-open="isStartModalOpen">
      <h2>Ingrese el valor de la Memoria y Kernel</h2>
      Memoria
      <input v-model="userMemory" type="text" />
      <br />
      <br />
      Kernel
      <input v-model="kernel" type="text" />
      <br />
      <br />
      <div class="ss-modal__actions">
        <button @click="onAceptStartData">Aceptar</button>
      </div>
      <br />
      <br />
    </Modal>
  </div>
</template>


<script setup>
import { reactive, ref } from "@vue/reactivity";
import { computed } from "@vue/runtime-core";
import Modal from "./components/Modal.vue";

const laptopValue = ref('');
const printerValue = ref('');
const emailValue = ref('');
const files = reactive([]);
const priorities = reactive([]);
const processType = ref("FCFS");
// la reactividad permite cambiar el valor de la constante y sus dependencias, accion en cadena, su referencia a la userMemory (proxi) no va a cambiar
const instructions = ref(null); // instrucciones del ch, ref datos nativos string, porque empieza null y cuando el usuario ingresa el valor, cambia a un array de instrucciones
const userMemory = ref(100);
const kernel = ref(19);
const isStartModalOpen = ref(true); //variable global para que el modal que active para pedir kernel y userMemory
//espacio reservado para el acumulador, reactive: es un conjunto de referencias reactivas (ref)
const memory = reactive([
  {
    type: "accumulator",
    value: 0,
  },
]);
const isStepByStep = ref(false);

//necesita ser asincrona, debido a que la reactividad impide ver los cambios al tener paso a paso activo
async function onExecute() {
  //para detener la reactividad por 1 milisegundo
  const delay = (ms) => new Promise((res) => setTimeout(res, ms));
  let instructionsLength = 0
  // loadsInfo permite saber saber el identificador del proceso 
  //fileIndex fichero de cada proceso
  for (let fileIndex = 0; fileIndex < loadsInfo.length; fileIndex++) {
    const loadInfo = loadsInfo[fileIndex];
    const startIndex = instructionsLength;
    instructionsLength += loadInfo.linesLength;
    //saber las instrucciones a que proceso pertenecen
    const instructionsSection = instructions.value?.slice(
      startIndex,
      instructionsLength,
    );
    for (let index = 0; index < instructionsSection.length; index++) {
      const instructionString = instructionsSection[index];
      if (isStepByStep.value) {
        await delay(1);
        alert(`Desea continuar? ${instructionString}`);
      }
      const parts = instructionString.split(" ");
      const instruction = parts[0];

      // funcion a ejecutar 
      const action = functions[instruction];
      if (action) {
        // debugger
        const newIndex = action(parts, index, fileIndex); //ejecutar la funcion
        if (newIndex === 'break') {
          break
        }
        if (newIndex !== null && newIndex !== undefined) {
          index = newIndex;
        }
      }
    }
  }
}

//esta funcion lee el archivo ch, $event nombre especial para identificarlo con event normal, reader objeto de la clase filereader nativo del navegador
function onLoadFile($event) {
  const reader = new FileReader();
  reader.readAsText($event.target.files[0]);
  reader.onload = () => {
    onFileSuccessLoad(reader.result);
  };
}

const loadsInfo = reactive([]);

const commentsLength = ref(0)

//carga los elementos del ch sin los comentarios
function onFileSuccessLoad(fileText) {
  // solicitar prioridad
  if (processType.value === "NO_EXPROPIATIVE") {
    const priority = parseInt(
      prompt(`De 0 a 100, agrege la prioridad del archivo`)
    );
    priorities.push({
      index: priorities.length,
      priority,
    });
  }

  const file = fileText.split("\n");
  commentsLength.value = file.filter(line => line.includes("//")).length;
  const linesWhitoutComments = file.filter(line => !line.includes("//"));
  const hasErrrors = checkChFileSintax(linesWhitoutComments);
  if (!hasErrrors) {
    if (instructions.value) {
      instructions.value = instructions.value.concat(linesWhitoutComments);
    } else {
      instructions.value = linesWhitoutComments;
    }
    loadsInfo.push({ linesLength: linesWhitoutComments.length });
    files.push(linesWhitoutComments);
  }
}

// diccionario de funciones validadoras, estas funciones sirven para verificar sintaxis
const validators = {
  nueva: checkNueva,
  cargue: checkCargue,
  almacene: checkAlmacene,
  lea: checkLea,
  sume: checkSume,
  reste: checkReste,
  multiplique: checkMultiplique,
  divida: checkDivida,
  potencia: checkPotencia,
  modulo: checkModulo,
  concatene: checkConcatene,
  elimine: checkElimine,
  extraiga: checkExtraiga,
  Y: checkY,
  O: checkO,
  NO: checkNO,
  muestre: checkMuestre,
  imprima: checkImprima,
  etiqueta: checkEtiqueta,
  vaya: checkVaya,
  vayasi: checkVayasi,
  retorne: checkRetorne,
};

//llama a los validators[intruction] para verificar sintaxis de la instruccion, si hay error muestra un alert
function checkChFileSintax(fileLines) {
  //comprueba si al menos un elemento del array cumple con la condición implementada por la función proporcionada.
  return fileLines.some((line, lineIndex) => {
    const parts = line.split(" ");
    const instruction = parts[0];
    //manda la instruccion porque es la llave del diccionario validators y el valor son parts
    if (validators[instruction]) {
      const errorMessage = validators[instruction](parts, fileLines, lineIndex);
      if (errorMessage) {
        alert(errorMessage);
        return errorMessage;
      }
    }
  });
}

function checkEtiqueta(parts, fileLines) {
  if (parts.length > 3) {
    return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`
  }
  
  const labels = [];
  const label = parts[1]
  const labelMatches = fileLines.filter((line) => {
    // instruccion array[0], labelName[1]
    const [instruccion, labelName] = line.split(' ')
    return instruccion === 'etiqueta' && labelName === label
  })

  // ? si retorna undefined, no hace nada, si esta repetido mas de una vez
  if (labelMatches?.length > 1) {
    return `Error, la etiqueta ${label} ya se encuentra registrada.`
  }

  const findedLabels = labels.filter(item => item === label)
}

function checkVaya(parts) {
  if (parts.length > 2) {
    return `Error, se estan utilizando mas de 2 operando en la operacion ${parts[0]}`;
  }
}
function checkNueva(parts) {
  if (parts[1] === parts[0]) {
    return `Error, se esta utilizado como nombre de la variable el mismo nombre de la operacion ${parts[0]}`;
  }
  else if (parts.length > 4) {
    return `Error, se estan utilizando mas de 4 operando en la operacion ${parts[0]}`;
  }
  else if (parts.length < 3) {
    return `Error, se estan utilizando menos de 3 operandos en la operacion ${parts[0]}`;
  }
  
  else if (!Object.keys(types).includes(parts[2])) {
    return `Error, el tipo de dato no es el correcto en la operacion ${parts[0]}`;
  }

  
}

function checkVayasi(parts) {
  if (parts.length !== 3) {
    return `Error, la etiqueta ${parts[0]} requiere se exactamente 3 operandos`;
  }
}

function checkCargue(parts) {
  if (parts.length > 2) {
    return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
  
}

function checkAlmacene(parts) {
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkLea(parts) {
  if (parts.length > 2) {
    return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkSume(parts) {
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkReste(parts) {
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkMultiplique(parts) {
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkDivida(parts) {
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkPotencia(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkModulo(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkConcatene(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkElimine(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkExtraiga(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkY(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkO(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkNO(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkMuestre(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkImprima(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}

function checkRetorne(parts) { 
  if (parts.length > 2) {
      return `Error, se estan utilizando mas de 2 operandos en la operacion ${parts[0]}`;
  }
}


// diccionario de las funciones de ejecución
const functions = {
  nueva: nuevaFunction,
  cargue: cargueFunction,
  almacene: almaceneFunction,
  lea: leaFunction,
  sume: sumeFunction,
  reste: resteFunction,
  multiplique: multipliqueFunction,
  divida: dividaFunction,
  potencia: potenciaFunction,
  modulo: moduloFunction,
  concatene: concateneFunction,
  elimine: elimineFunction,
  extraiga: extraigaFunction,
  Y: YFunction,
  O: OFunction,
  NO: NOFunction,
  muestre: muestreFunction,
  imprima: imprimaFunction,
  vaya: vayaFunction,
  vayasi: vayasiFunction,
  retorne: retorneFunction,
  email: emailFunction,
};

//agregamos el valor de las variables al diccionario de variables, a partir de la destructuracion
//El separador de coma se utiliza para omitir valores en un arreglo.
function nuevaFunction(parts, index, fileIndex) {
  const [, name] = parts; //array partes se compone de partes de la instruccion, nombre de la variable
  const variableValue = addDefaultValue(parts);
  mainMemoryVariables[name + fileIndex] = variableValue;
}

function leaFunction([, variableName], index, fileIndex) {
  let newValue = prompt(`Agregue el valor de ${variableName}`); //en el input aparece el valor de la variable (nombre de la variable90)
  if (newValue) {
    newValue = parseInt(newValue);
  }
  if (newValue !== "") {
    mainMemoryVariables[variableName + fileIndex] = newValue;
  }
}

//Cárguese/copie en el acumulador el valor almacenado en la variable indicada por el operando.
function cargueFunction(parts, index, fileIndex) {
  const variableName = parts[1];
  const variableValue = mainMemoryVariables[variableName + fileIndex];
  mainMemory.value[0].value = variableValue; //lo primero del array es el acumulador
}

// //Si el valor del acumulador es mayor a cero, va a la instrucción que corresponde a
// la etiqueta indicada por el primer operando.
// Si el valor del acumulador es menor a cero, va la instrucción que corresponde a la
// etiqueta indicada por el segundo operando
// o Si el acumulador es cero a la siguiente instrucción adyacente a la instrucción
// vayasi y siga la ejecución a partir de allí.
function vayasiFunction([, label1, label2], index) {
  let newIndex = null;
  const accumulator = mainMemory.value[0].value;
  if (accumulator > 0) {
    const indexToGo = mainMemoryLabels.value[label1] - commentsLength.value -1;
    newIndex = indexToGo;
    // debugger;
  } else if (accumulator < 0) {
    const indexToGo = mainMemoryLabels.value[label2] - commentsLength.value -1;
    newIndex = indexToGo;
    // debugger;
  }
  // debugger;
  return newIndex;
}

//Decremente el acumulador en el valor indicado por la variable que señala el operando.
// estar en la posicion 1 de ese array variables
function resteFunction([, variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const subtractValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue - subtractValue;
  mainMemory.value[0].value = newValue;
}

//Incremente el valor del acumulador en el valor indicado por la variable señalada por el operando.
function sumeFunction([, variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const sumValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue + sumValue;
  mainMemory.value[0].value = newValue;
}

//Guarde/copie el valor que hay en el acumulador a la variable indicada por el operando.
function almaceneFunction([, variableName], index, fileIndex) {
  mainMemoryVariables[variableName + fileIndex] = mainMemory.value[0].value;
}

//Multiplique el valor del acumulador por el valor indicado por la variable señalada por el operando.
function multipliqueFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const multiplyValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue + multiplyValue;
  mainMemory.value[0].value = newValue;
}

//Divida el valor del acumulador por el valor indicado por la variable señalada por el operando. El divisor deberá ser una cantidad diferente de cero.
function dividaFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const divValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue + divValue;
  mainMemory.value[0].value = newValue;
}

//Eleve el acumulador a la potencia señalada por el operando(los exponentes pueden ser valores enteros, positivos o negativos)
function potenciaFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const powValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue ^ powValue;
  mainMemory.value[0].value = newValue;
}

//Obtenga el modulo al dividir el valor del acumulador por el valor indicado por la variable señalada por el operando.
function moduloFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const modValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue % modValue;
  mainMemory.value[0].value = newValue;
}

//Genere una cadena que una la cadena dada por el operando a la cadena que hay en el acumulador (Operando alfanumérico)
function concateneFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const stringValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue.concat(' ', stringValue);
  mainMemory.value[0].value = newValue;
}

//Genere una subcadena que elimine cualquier aparición del conjunto de caracteres dados por el operando de la cadena que se encuentra en el acumulador (operando alfanumérico)
function elimineFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const deleteValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue.filter(function(item) {
    return item !== deleteValue;
  })
}

//Genere una subcadena que extraiga los primeros caracteres (dados por el valor numérico operando) de la cadena que se encuentra en el acumulador (operando numérico)
function extraigaFunction([,variableName], index, fileIndex) {
  const currentValue = mainMemory.value[0].value;
  const extractValue = mainMemoryVariables[variableName + fileIndex];
  const newValue = currentValue.filter(function(item) {
    return item !== extractValue
  })
}

//Produce una operación lógica Y (AND) entre el primer operando y el segundo operando que son variables lógicas y la almacena en el tercer operando.
function YFunction([,valueA, valueB, variableName], index, fileIndex) {
  mainMemoryVariables[variableName] = valueA && valueB;
}

// roduce una operación lógica O (OR) entre el primer operando y el segundo operando que son variables lógicas y la almacena en el tercer operando.
function OFunction([,valueA, valueB, variableName], index, fileIndex) {
  mainMemoryVariables[variableName] = valueA || valueB;
}

//Produce una operación de negación lógica para el primer operando que es una variable lógica y el resultado se almacena en el segundo operando.
function NOFunction([,valueA, variableName], index, fileIndex) {
  mainMemoryVariables[variableName] = !valueA;
}

// Presente por pantalla el valor que hay en la variable indicada por el operando, si el operando es acumulador muestre el valor del acumulador.
function muestreFunction([,valueName], index, fileIndex) {
  const valueToAdd = mainMemoryVariables[valueName + fileIndex]
  laptopValue.value += valueToAdd
}

// Lo mismo que el anterior pero presentándolo en la impresora
function imprimaFunction([,valueName], index, fileIndex) {
  const valueToAdd = mainMemoryVariables[valueName + fileIndex]
  printerValue.value += valueToAdd
}

function emailFunction([,valueName], index, fileIndex) {
  const valueToAdd = mainMemoryVariables[valueName + fileIndex]
  emailValue.value += valueToAdd
}

// El programa termina; debe ser la última instrucción del programa y tiene opcionalmente un operando numérico entero 
function retorneFunction(){
  return 'break'
}

// Salte a la instrucción que corresponde a la etiqueta indicada por el operando y siga la ejecución a partir de allí.
function vayaFunction([,label1], index, fileIndex){
  let newIndex = null;
  const indexToGo = mainMemoryLabels.value[label1]  - commentsLength.value -1;
  return indexToGo;
}

//diccionario, objeto js para los tipos de variable
const types = {
    I: 0,
    L: 0,
    R: 0,
    C: "",
};

//De la instruccion del archivo ch se obtiene todo lo que diga nueva, se agrega variables, variable reactiva
const variables = computed(() => {
  const variables = [];
  loadsInfo.forEach((info, fileIndex) => {
    const values = instructions.value?.slice(0, info.linesLength) || [];
    values.forEach((instruction) => {
      if (instruction && instruction.includes("nueva")) {
        //instruction debe existir
        const parts = instruction.split(" "); //obtiene el array de la separacion por espacios del archivo ch
        variables.push({
          type: "variable",
          value: addDefaultValue(parts),
          name: parts[1],
          fileIndex,
        });
      }
    });
  });
  return variables;
});

//agregar el valor de las variables
function addDefaultValue(parts) {
  const value = parts[3];
  const type = parts[2];
  
  //si no hay valor en la variable el fichero ch, retorna el valor por defecto, de los contrario retorna el valor que traiga (original)
  let valueToReturn = value ?? types[type];
  if (["I", "R"].includes(type)) valueToReturn = Number(valueToReturn);
  return valueToReturn;
}

const variableNames = computed(() => {
  return mainMemory.value.reduce((variables, memoryItem, index) => {
    if (memoryItem.type === "variable") {
      variables.push({ name: memoryItem.name, index });
    }
    return variables;
  }, []);
});

function formatIndex(index) {
  let formattedIndex = index;
  if (index < 10) formattedIndex = `0${formattedIndex}`;
  if (index < 100) formattedIndex = `0${formattedIndex}`;
  if (index < 1000) formattedIndex = `0${formattedIndex}`;
  return formattedIndex;
}

function formatInstructionIndex(index) {
  const indexValue = Number(index) + kernel.value + 1;
  return formatIndex(indexValue);
}

function onStart() {
  isStartModalOpen.value = false;
  for (let index = 0; index < kernel.value; index++) {
    memory.push({ type: "kernel", value: "CH Maquina" });
  }
}

function onAceptStartData() {
  if (
    userMemory.value <= 0 ||
    kernel.value <= 0 ||
    userMemory.value > 9999 ||
    userMemory.value <= kernel.value
  ) {
    alert(
      "Datos incorrectos, la userMemory debe ser menor a 9999 posiciones y el tamaño del kernel debe ser menor al tamaño de la userMemory"
    );
  } else if (!userMemory.value || isNaN(Number(userMemory.value))) {
    alert(
      "Datos incorrectos, el dato ingresado es obligatorio y debe ser un digito"
    );
  } else {
    onStart();
  }
}

//diccionario mainMemory
//una propiedad computada es la suma de las propiedades reactivas, es de solo lectura, no se debe modificar
const mainMemory = computed(() => {
  let result = [...memory];
  loadsInfo.forEach((loadInfo, fileIndex) => {
    const chInstructions =
      instructions.value?.slice(0, loadInfo.linesLength) || []; //carga de instrucciones ch
    const formattedInstructions = chInstructions.map((instruction) => ({
      //separa el string por tipo y por valor
      type: "instruction",
      value: instruction,
    }));

    const variablesInFile =
      variables.value.filter(
        (memoryItem) => memoryItem.fileIndex === fileIndex
      ) || [];

    result = result.concat(
      //concatena las instruccion ch con las variables que van a ir cambiando
      formattedInstructions.concat(variablesInFile)
    );
  });

  for (let index = result.length; index < userMemory.value; index++) {
    result.push({ type: "empty", value: "" });
  }

  return result;
});

// metodo para almancenar etiquetas
const mainMemoryLabels = computed(() => {
  return mainMemory.value.reduce((labels, memoryItem) => {
    if (
      memoryItem.type === "instruction" &&
      memoryItem.value.includes("etiqueta")
    ) {
      // en un array se almacena toda la informacion de la etiqueta, nombre y valor 
      const [, name, value] = memoryItem.value.split(" ");
      //array labels con ese nombrele asigna el valor de dicha etiqueta
      labels[name] = parseInt(value);
    }
    return labels;
  }, {});
});

// metodo para almancenar el array de etiquetas en el diccionario de etiquetas, solo lectura
const labels = computed(() => {
  // es metodo que sirve para almacenar en un tipo de dato, objeto, array, los datos que vienen de otro array, al ponerle una condicion
  return mainMemory.value.reduce((labels, memoryItem) => {
    if (
      memoryItem.type === "instruction" &&
      memoryItem.value.includes("etiqueta")
    ) {
      const [, labelName, labelValue] = memoryItem.value.split(" ");
      const labelIndex = formatInstructionIndex(labelValue - 1);
      labels.push({ name: labelName, index: labelIndex });
    }
    return labels;
  }, []);
});

//crea como un objeto vacio, no necesita acceder al atributo value, menos nativo, mas alto nivel
const mainMemoryVariables = reactive({});

//crea metodo para la planificacion de procesos por prioridad no expropiativo
const noExpropiativeOrder = computed(() => {
  const array = [];
  if (files.length) {
    array.push(files[0]);

    const [firstPriority, ...otherPiorities] = priorities;
    const sortedPriorities = otherPiorities.sort(
      (a, b) => b.priority - a.priority
    );
    sortedPriorities.forEach((priority) => {
      array.push(files[priority.index]);
    });
  }
  return array;
});
</script>

<style>
.ss-menu__container {
  display: flex;
}

.ss-page {
  margin-top: 44px;
}

.ss-menu {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 44px;
}

.ss-header {
  display: flex;
  justify-content: space-between;
}

.ss-accumulator {
  border: 1px solid black;
  padding: 8px;
  border-radius: 8px;
}

.ss-laptop {
  position: relative;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.ss-laptop__value {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  top: 52px;
  font-size: 20px;  
}

.ss-tables {
  display: grid;
  grid-template-columns: 1fr 260px 1fr;
  grid-auto-rows: auto auto auto;
}

.ss-table__container {
  border: 1px solid black;
  padding: 8px;
}

.ss-file-submenu {
  position: relative;
}

.ss-file-submenu__actions {
  position: absolute;
  top: 20px;
  left: 0;
}

.ss-file-submenu .ss-file-submenu__actions {
  display: none;
}

.ss-file-submenu:hover .ss-file-submenu__actions {
  display: block;
}
</style>
