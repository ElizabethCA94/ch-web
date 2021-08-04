<template>
  <div class="ss-page">
    <p>
      Variables <br />
       <!-- Texto preformateado con saltos de linea -->
      <pre>{{ mainMemoryVariables }}</pre>
    </p>
    <p>
      Labels <br />
      <!-- Texto preformateado con saltos de linea -->
      <pre>{{ mainMemoryLabels }}</pre>
    </p>
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

    <div class="ss-header">
      <h1>CH Máquina</h1>
      <div class="ss-accumulator">
        <!--Referencia reactiva: ref, proxy reflectivo, es una referencia en memoria de un valor global
         mientras de reactive: es un conjunto de referencias reactivas  -->
        <h2>Acumulador {{ mainMemory[0].value }}</h2>
      </div>
    </div>

    <div class="ss-tables">
      <div class="ss-table__container ss-variables-table">
        <h2>Variables</h2>
        <table>
          <tr
            v-for="(variable, variableIndex) in variableNames"
            :key="variableIndex"
          >
            <td>{{ formatIndex(variable.index) }} {{ variable.name }}</td>
          </tr>
        </table>
      </div>
      <div class="ss-laptop">
        <p>{{ isStepByStep ? "paso a paso" : "" }}</p>
        <img src="computadora.png" />
        <p>{{ isStepByStep ? "modo usuario" : "modo kernel" }}</p>
      </div>
      <div class="ss-table__container ss-labels-table">
        <h2>Etiquetas</h2>
        <table>
          <tr v-for="(label, labelIndex) in labels" :key="labelIndex">
            <td>{{ label.index }} {{ label.name }}</td>
          </tr>
        </table>
      </div>

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
            <td>{{ formatIndex(memoryIndex) }} {{ memoryItem.value }}</td>
          </tr>
        </table>
      </div>
    </div>
    <!-- is open es una propiedad de entrada definida del componenete modal  -->
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

// la reactividad permite cambiar el valor de la constante y sus dependencias, accion en cadena, su referencia a la userMemory (proxi) no va a cambiar
const instructions = ref(null); // instrucciones del ch, ref datos nativos string, porque empieza null y cuando el usuario ingresa el valor, cambia a un array de instrucciones
const userMemory = ref(100);
const kernel = ref(19);
const isStartModalOpen = ref(true); //variable global para que el modal que active para pedir kernel y userMemory
// kernel.value = window.prompt('Ingrese el valor del Kernel')
//espacio reservado para el acomulador y kernel
const memory = reactive([
  {
    type: "accumulator",
    value: 0,
  },
]);
const isStepByStep = ref(false);

//necesita ser asincrona, debido a que la reactividad impide ver los cambios al tener paso a paso activo
async function onExecute() {
  const delay = (ms) => new Promise((res) => setTimeout(res, ms));
  for (let index = 0; index < instructions.value.length; index++) {
    const instructionString = instructions.value[index];
    if (isStepByStep.value) {
      await delay(1);
      alert(`Desea continuar? ${instructionString}`);
    }
    const parts = instructionString.split(" ");
    const instruction = parts[0];
    const action = functions[instruction];
    if (action) {
      action(parts, index); //ejecutar la funcion
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

//carga los elementos del ch sin los comentarios
function onFileSuccessLoad(fileText) {
  const file = fileText.split("\n");
  const linesWhitoutComments = file.filter((line) => !line.includes("//"));
  const hasErrrors = checkChFileSintax(linesWhitoutComments);
  if (!hasErrrors) {
    instructions.value = linesWhitoutComments;
  }
}

// diccionario de funciones validadoras
const validators = {
  nueva: checkNueva,
  cargue: () => {},
  almacene: () => {},
  lea: () => {},
  sume: () => {},
  reste: () => {},
  multiplique: () => {},
  divida: () => {},
  potencia: () => {},
  modulo: () => {},
  concatene: () => {},
  elimine: () => {},
  extraiga: () => {},
  Y: () => {},
  O: () => {},
  NO: () => {},
  muestre: () => {},
  imprima: () => {},
  etiqueta: () => {},
  vaya: () => {},
  vayasi: () => {},
  retorne: () => {},
};

// diccionario de las funciones de ejecución
const functions = {
  nueva: nuevaFunction,
  cargue: cargueFunction,
  almacene: almaceneFunction,
  lea: leaFunction,
  sume: sumeFunction,
  reste: resteFunction,
  multiplique: () => {},
  divida: () => {},
  potencia: () => {},
  modulo: () => {},
  concatene: () => {},
  elimine: () => {},
  extraiga: () => {},
  Y: () => {},
  O: () => {},
  NO: () => {},
  muestre: () => {},
  imprima: () => {},
  etiqueta: () => {},
  vaya: () => {},
  vayasi: vayasiFunction,
  retorne: () => {},
};

//agregamos el valor de las variables a al diccionario de variables, a partir de la destructuracion
function nuevaFunction(parts) {
  const [, name] = parts;
  const variableValue = addDefaultValue(parts);
  mainMemoryVariables[name] = variableValue;
}

//permitimos que el usuario pueda agregar un nuevo valor a la variable y si no lo desea se deja el valor por defecto
function leaFunction([, variableName]) {
  const newValue = prompt(`Agregue el valor de ${variableName}`);
  if (newValue !== "") {
    mainMemoryVariables[variableName] = newValue;
  }
}

//cambia el valor del acomulador, al de la resta del acomulador - valor indicado
function cargueFunction(parts) {
  const variableName = parts[1];
  const variableValue = mainMemoryVariables[variableName];
  mainMemory.value[0].value = variableValue;
}

function vayasiFunction([, label1, label2], index) {
  const accumulator = mainMemory.value[0].value;
  if (accumulator > 0) {
    index = mainMemoryLabels[label1];
  } else if (accumulator > 0) {
    index = mainMemoryLabels[label2];
  }
}

function resteFunction([, variableName]) {
  const currentValue = mainMemory.value[0].value;
  const subtractValue = mainMemoryVariables[variableName];
  const newValue = currentValue - subtractValue;
  mainMemory.value[0].value = newValue;
}

function sumeFunction([, variableName]) {
  const currentValue = mainMemory.value[0].value;
  const sumValue = mainMemoryVariables[variableName];
  const newValue = currentValue + sumValue;
  mainMemory.value[0].value = newValue;
}

function almaceneFunction([, variableName]) {
  mainMemoryVariables[variableName] = mainMemory.value[0].value;
}

function checkChFileSintax(fileLines) {
  return fileLines.some((line) => {
    const parts = line.split(" ");
    const instruction = parts[0];
    //manda la instruccion porque es la llave del diccionario validators
    if (validators[instruction]) {
      const errorMessage = validators[instruction](parts);
      if (errorMessage) {
        alert(errorMessage);
        return errorMessage;
      }
    }
  });
}

function checkNueva(parts) {
  if (parts.length > 4 || parts.length < 3) {
    return `Error, se estan utilizando mas de 4 operandos ó menos de 3 operandos en la operacion ${parts[0]}`;
  }
}

//De instruccion toodo lo que diga nueva, se toma de la variable reactiva instrucciones,
const variables = computed(() => {
  const variables = [];
  const values = instructions.value || [];
  values.forEach((instruction) => {
    if (instruction && instruction.includes("nueva")) {
      const parts = instruction.split(" "); //obtiene el array de la separacion por espacios del archivo ch
      variables.push({
        type: "variable",
        value: addDefaultValue(parts),
        name: parts[1],
      });
    }
  });
  return variables;
});

//agregar el valor de las variables
function addDefaultValue(parts) {
  const value = parts[3];
  const type = parts[2];
  const types = {
    I: 0,
    L: 0,
    R: 0,
    C: "",
  };
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
  const chInstructions = instructions.value || []; //carga de instrucciones ch
  const formattedInstructions = chInstructions.map((instruction) => ({
    //separa el string por tipo y por valor
    type: "instruction",
    value: instruction,
  }));

  //concatena las instruccion ch con las variables que van a ir cambiando 
  const result = memory
    .concat(formattedInstructions)
    .concat(variables.value || []);

  for (let index = result.length; index < userMemory.value; index++) {
    result.push({ type: "empty", value: "" });
  }

  return result;
});

const mainMemoryLabels = computed(() => {
  return mainMemory.value.reduce((labels, memoryItem) => {
    if (
      memoryItem.type === "instruction" &&
      memoryItem.value.includes("etiqueta")
    ) {
      const [, name, value] = memoryItem.value.split(" ");
      labels[name] = parseInt(value);
    }
    return labels;
  }, {});
});

const labels = computed(() => {
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
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
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
