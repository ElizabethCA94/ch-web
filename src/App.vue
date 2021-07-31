<template>
  <div class="ss-page">
    <div class="ss-menu">
      <span class="ss-file-submenu">
        <button>File</button>
        <div class="ss-file-submenu__actions">
          <!-- accion del usuario al ingresar datos en el input -->
          <input type="file" @input="onLoadFile" />
        </div>
      </span>
    </div>
    <h1>CH Máquina</h1>

    <div class="ss-tables">
      <div class="ss-table ss-variables-table">
        <table>
          <tr
            v-for="(variable, variableIndex) in variables"
            :key="variableIndex"
          >
            <td>{{ variable }}</td>
          </tr>
        </table>
      </div>
      <div class="ss-table ss-labels-table">dos</div>
      <div class="ss-table ss-file-table">
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
      <div class="ss-table ss-memory-table">
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
const instructions = ref(null); //ref datos nativos string, porque empieza null y cuando el usuario ingresa el valor, cambia a un array de instrucciones
const userMemory = ref(100);
const kernel = ref(19);
const isStartModalOpen = ref(true); //variable global para que el modal que active para pedir kernel y userMemory
// kernel.value = window.prompt('Ingrese el valor del Kernel')
const memory = reactive([
  {
    type: "accumulator",
    value: 0,
  },
]);

//esta funcion lee el archivo ch, $event nombre especial para identificarlo con event normal, reader objeto de la clase filereader nativo del navegador
function onLoadFile($event) {
  const reader = new FileReader();
  reader.readAsText($event.target.files[0]);
  reader.onload = () => {
    onFileSuccessLoad(reader.result);
  };
}

function onFileSuccessLoad(fileText) {
  const fileLines = fileText.split("\n");
  instructions.value = fileLines.filter((line) => !line.includes("//"));
}

/*if(instrucciones[0].lower()=="nueva"):
            if(len(instrucciones)==3):
                if(instrucciones[2]=="I"):
                    instrucciones.append("0")
                elif(instrucciones[2]=="L"):
                    instrucciones.append("0")
                elif(instrucciones[2]=="R"):
                    instrucciones.append("0")
                elif(instrucciones[2]=="C"):
                    instrucciones.append(" ")
            variables[llave] = { 'tipo': instrucciones[2], 'valor': instrucciones[3] }*/
const variables = computed(() => {
  const variables = [];
  const values = instructions.value || [];
  values.forEach(instruction => {
    if (instruction && instruction.includes("nueva")) {
      const parts = instruction.split(' ')
      variables.push({ type: 'variable', value: addDefaultValue(parts), name: parts[1] });
    }
  });
  return variables;
});

function addDefaultValue(parts) {
  const types = {
    'I': 0,
    'L': 0,
    'R': 0,
    'C': '',
  }
  if (!parts[3]) return types[2]
  return parts[3]
}

// const variableNames = computed(() => {
//   re
// })

function formatIndex(index) {
  let formattedIndex = index;
  if (index < 10) formattedIndex = `0${formattedIndex}`;
  if (index < 100) formattedIndex = `0${formattedIndex}`;
  if (index < 1000) formattedIndex = `0${formattedIndex}`;
  return formattedIndex;
}

function formatInstructionIndex(index) {
  const indexValue = index + kernel.value + 1;
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

//una propiedad computada es la suma de las propiedades reacitivas, es de solo lectura, no se debe modificar
const mainMemory = computed(() => {
  const chInstructions = instructions.value || []; //carga de instrucciones ch
  const formattedInstructions = chInstructions.map((instruction) => ({
    //separa el string por tipo y por valor
    type: "instruction",
    value: instruction,
  }));

  const result = memory.concat(formattedInstructions).concat(variables.value || []);

  for (let index = result.length; index < userMemory.value; index++) {
    result.push({ type: "empty", value: "" });
  }

  return result;
});
</script>

<style>
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

.ss-tables {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-auto-rows: auto auto;
}

.ss-table {
  border: 1px solid black;
}

.ss-file-submenu .ss-file-submenu__actions {
  display: none;
}

.ss-file-submenu:hover .ss-file-submenu__actions {
  display: block;
}
</style>
