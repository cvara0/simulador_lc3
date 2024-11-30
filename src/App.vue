<script setup>
import { ref, reactive, onMounted, computed,onUnmounted, watch } from "vue"
import Registros from "./components/Registros.vue"



const registros = ref([])
const memoriaRam = ref(new Int16Array(32768))

const aBuscar = ref('')
const errores = ref([])
const errorIns = ref(false)
const cantIns = ref(0)
const signo = ref("")


const insTextoHexa = reactive({
  texto:`1265
14A3
1642
5642
5664
96FF
33FF
29FE
6A84
7643
1CA0
1DBF
07FE
1CA0
1DBF
03FE
1CE0
1DA1
0DFE
1CE0
1DA1
09FE
C040`,
})

const inicio = ref("3000")
const desplazamiento = ref(12285)
const pc = ref(12288)

const scrollDelta = ref(0)

onMounted(() => {
  limpiarMemoria()
  limpiarRegistros()
  //window.addEventListener('wheel', handleWheel)
})

onUnmounted(() => {
      window.removeEventListener('wheel', handleWheel)
    })

const limpiarMemoria = () => {
  errorIns.value = false
  memoriaRam.value = new Int16Array(32768)
}
const limpiarRegistros = () => {
  registros.value = new Array(8).fill("0000")
}

const handleWheel = (event) => {
      scrollDelta.value = event.deltaY
      if(Math.sign(event.deltaY) == 1){
        desplazamiento.value++
      }else{
        desplazamiento.value--
      }
      desplazamiento.value = desplazamiento.value <= 0 ? 0 : desplazamiento.value > 16384 ? 16384 : desplazamiento.value
      event.preventDefault();
    }

watch(aBuscar, ()=>{
  if(aBuscar.value)
      desplazamiento.value = hexadecimalADecimalConSigno(aBuscar.value)
  else
    desplazamiento.value = pc.value - 3 
})


///////////////////////////////////////////////////////////////////////////////
const cargar = () => {
  let ini = hexadecimalADecimalSinSigno(inicio.value);
  errorIns.value = false
  const lineas = insTextoHexa.texto
    .trim()
    .split("\n")
    .map((linea) => linea.trim())
 
  errores.value = lineas
    .map((linea, index) => ({ linea: index + 1, valida: /^[0-9A-Fa-f]{4}$/.test(linea) }))
    .filter((r) => !r.valida)

  if (errores.value.length === 0) {
    
    lineas.forEach((valor, i) => {
      memoriaRam.value[ini + i] = hexadecimalADecimalConSigno(valor)
    })
    pc.value = ini
    desplazamiento.value = pc.value >= 0 && pc.value < 3 ? pc.value : pc.value - 3
  }

}

///////////////////////////////////////////////////////////////////////////////////
const recorrer = (esRecorrer) => {
  
  
  /* for (let i = 0; i < contarIns; i++) {
   
  } */
//seguir con ejecutar todo,contar lineas, subir archivo cargar en textarea drag and drop, ver lenguaje assembly
  
  if (memoriaRam.value[pc.value] === 0) {
    errorIns.value = true
  } else {
    let insBin = decimalABinarioC2(memoriaRam.value[pc.value])
    const ins = {
      opcode: insBin.substring(0, 4),
      dr: parseInt(insBin.substring(4, 7), 2),
      sr: parseInt(insBin.substring(4, 7), 2),
      sr1: parseInt(insBin.substring(7, 10), 2),
      baseR: parseInt(insBin.substring(7, 10), 2),
      bitImm5: parseInt(insBin.substring(10, 11), 2),
      imm5: binarioC2ADecimal(insBin.substring(11, 16)),
      sr2: parseInt(insBin.substring(13, 16), 2),
      PCoffset9: binarioC2ADecimal(insBin.substring(7, 16)),
      PCoffset11: binarioC2ADecimal(insBin.substring(5, 16)),
      n: insBin[4],
      z: insBin[5],
      p: insBin[6],
      offset6: binarioC2ADecimal(insBin.substring(10, 16))
    }

    registros.value = registros.value.map((i) => hexadecimalADecimalConSigno(i))

    if (pc.value > 16384) 
      pc.value = 16384
    else 
      {
        pc.value++
        desplazamiento.value = pc.value - 3 //ver para 0000
      }

    switch (ins.opcode) {
      case "0001": //add
        registros.value[ins.dr] = registros.value[ins.sr1] + (ins.bitImm5 === 0 ? registros.value[ins.sr2] : ins.imm5);
        signo.value = Math.sign(registros.value[ins.dr]);
        break;
      case "0101": //and
        registros.value[ins.dr] = registros.value[ins.sr1] & (ins.bitImm5 === 0 ? registros.value[ins.sr2] : ins.imm5);
        signo.value = Math.sign(registros.value[ins.dr]);
        break;
      case "1001": //not
        registros.value[ins.dr] = ~registros.value[ins.sr1];
        signo.value = Math.sign(registros.value[ins.dr]);
        break;
      case "0000": //br
        if (
          (ins.n === "1" && signo.value === -1) ||
          (ins.z === "1" && signo.value === 0) ||
          (ins.p === "1" && signo.value === 1)
        )
          pc.value = pc.value + ins.PCoffset9
          desplazamiento.value = pc.value >= 0 && pc.value < 3 ? pc.value : pc.value - 3
        break;
      case "1100": //jmp ret
        pc.value = registros.value[ins.baseR];
        desplazamiento.value = pc.value >= 0 && pc.value < 3 ? pc.value : pc.value - 3
        break;
      case "0100": //jsr jsrr
        let temp = pc.value;
        pc.value =
          parseInt(insBin.substring(4, 5), 2) === 0
            ? registros.value[ins.baseR]
            : pc.value + ins.PCoffset11;
        registros.value[7] = temp;
        desplazamiento.value = pc.value >= 0 && pc.value < 3 ? pc.value : pc.value - 3
        break;
      case "0010": //ld
        registros.value[ins.dr] = memoriaRam.value[pc.value + ins.PCoffset9];
        break;
      case "1010": //ldi
        registros.value[ins.dr] = memoriaRam.value[memoriaRam.value[pc.value + ins.PCoffset9]];
        break;
      case "0110": //ldr
        registros.value[ins.dr] = memoriaRam.value[registros.value[ins.baseR] + ins.offset6];
        break;
      case "1110": //lea
        registros.value[ins.dr] = pc.value + ins.PCoffset9;
        break;
      case "0011": //st
        memoriaRam.value[pc.value + ins.PCoffset9] = registros.value[ins.sr];
        break;
      case "1011": //sti
        memoriaRam.value[memoriaRam.value[pc.value + ins.PCoffset9]] = registros.value[ins.sr];
        break;
      case "0111": //str
        memoriaRam.value[registros.value[ins.baseR] + ins.offset6] = registros.value[ins.sr];
        break;
      default:
        errorIns.value = true
        break;
    }

    registros.value = registros.value.map((i) => decimalConSignoAHexadecimal(i))
  }

}

////////////////////////////////////////////////////////////////////////////////



///////////////////////////////////////////////////////////////////////
function binarioC2ADecimal(binario) {
  const longitud = binario.length;
  const esNegativo = binario[0] === "1"; // Chequear el bit de signo

  if (esNegativo) {
    // Calcular complemento a dos
    const valorPositivo = (~parseInt(binario, 2) + 1) & ((1 << longitud) - 1);
    return -valorPositivo;
  } else {
    // Convertir directamente si es positivo
    return parseInt(binario, 2);
  }
}

const decimalABinarioC2 = (decimal) => {
  const mask = (1 << 16) - 1;
  return (decimal & mask).toString(2).padStart(16, '0');
}

function decimalConSignoAHexadecimal(sdecimal) {
  if (sdecimal >= 0) {
    return sdecimal.toString(16).toUpperCase().padStart(4, '0')
  } else {
    return (Math.pow(2, 16) + sdecimal).toString(16).toUpperCase().padStart(4, '0')
  }
}

function decimalSinSignoAHexadecimal(udecimal){
    return udecimal.toString(16).toUpperCase().padStart(4, '0')
  }

function hexadecimalADecimalSinSigno(hex){
  return parseInt(hex, 16)
}

function hexadecimalADecimalConSigno(hex) {
  // Convertir de hexadecimal a decimal sin signo
  const valorSinSigno = parseInt(hex, 16);
  // Calcular el límite para números positivos
  const limiteSigno = Math.pow(2, 16 - 1);

  // Si el valor supera o iguala al límite, es negativo
  if (valorSinSigno >= limiteSigno) {
    return valorSinSigno - Math.pow(2, 16);
  } else {
    return valorSinSigno;
  }
}


const reiniciarPc = computed(()=>{
  errorIns.value = false
  pc.value = hexadecimalADecimalConSigno(inicio.value)
  desplazamiento.value = pc.value < 3 ? pc.value :pc.value - 3
})

//const indiceRecorrer = computed(() => decimalConSignoAHexadecimal(pc.value).padStart(4, "0") ) 


const contarIns = computed(() => (insTextoHexa.texto.trim() === '') ? 0 : insTextoHexa.texto.split('\n').filter(line => line.trim() !== '').length)


////////////////////////////////////////////////////////////////////////////////


const ramDecimalAHexa = computed(() => decimalConSignoAHexadecimal(memoriaRam.value[pc.value]))
  

///////////////////////////////////////////////////////////////////////
    // Función para actualizar el valor en memoria
const updateMemory = (index, newValue) => {
      // Convierte el valor hexadecimal a decimal antes de guardarlo
      const decimalValue = parseInt(newValue, 16) || 0
      memoriaRam.value[index] = decimalValue
    }

////////////////////////////////////////////////////////////////////////////////////


function handleFileDrop(event) {
      const file = event.dataTransfer.files[0]; // Obtener el archivo
      if (file && file.type === 'text/plain') {
        const reader = new FileReader();
        reader.onload = (e) => {
          processFileContent(e.target.result); // Procesar el contenido del archivo
        };
        reader.readAsText(file); // Leer el archivo como texto
      }
    }
function processFileContent(content) {
      fileContent = content; // Asignar el contenido del archivo al textarea

      // Si deseas recorrer cada línea del archivo
      const lines = content.split('\n');
      lines.forEach((line, index) => {
        console.log(`Línea ${index + 1}: ${line}`);
      });
    }
  

</script>

<template>
  <div class="animate__animated  animate__fadeInDown">
    <h5>Laboratorio 5</h5>
    <h1>🖳Simulador de ISA de la LC3</h1>
    <h5>(Basado en el laboratorio 2)</h5>
  </div>
 
  <div class="container mt-3 animate__animated animate__fadeIn" >
    <div class="row justify-content-center">
      <div class="col-12 col-sm-12 col-md-5 col-lg-4 col-xl-4 col-xxl-4 animate__animated animate__fadeInLeft">
        <h4 class="mt-4">Instrucciones en Hexadecimal</h4>

        <form @submit.prevent="cargar" class="form-floating d-grid gap-2">

          <div class="form-floating mb-2">
            <input v-model="inicio" maxlength="4" type="text" class="form-control" id="floatingInput" placeholder="name@example.com">
            <label for="floatingInput">Dirección de inicio:</label>
          </div>

          <textarea
            v-model="insTextoHexa.texto"
            class="numbered"
            style="height: 400px"
    
          > </textarea>
          <h5>Cantidad de instrucciones: {{contarIns}}</h5>
          <button type="submit" class="btn btn-primary">Cargar a memoria ⤴</button>
        </form>
        
        <div v-if="errores.length > 0" class="mt-2">
          <div class="alert alert-danger" role="alert">Errores de carga:</div>
          <ul v-for="item in errores" class="list-group">
            <li class="list-group-item">En linea {{ item.linea }}</li>
          </ul>
        </div>
        <div v-else-if="memoriaRam.some((i) => i !== 0)" class="mt-2">
          <div class="alert alert-success" role="alert">Carga OK👍</div>
        </div>
      </div>

      <div class="col-12 col-sm-6 col-md-6 col-lg-4 col-xl-4	col-xxl-4  animate__animated animate__fadeInUp">
        <h4>Memoria RAM {{ memoriaRam.length }} </h4>
        <div class="row justify-content-between">
          <button @click="limpiarMemoria" type="button" class="col btn btn-danger m-2">
            Limpiar memoria
          </button>
          <button @click="reiniciarPc" type="button" class="col btn btn-danger m-2">
            Reiniciar PC
          </button>
  
    
          <div class="form-floating mb-2">
            <input v-model="aBuscar" type="text" maxlength="4" class="form-control" id="floatingInput" placeholder="name@example.com">
            <label for="floatingInput">🔍Buscar dirección:</label>
          </div>
        </div>
        
        <div v-show="errorIns" class="alert alert-danger" role="alert">
          ⛔  Ejecución detenida  ⛔
        </div>
        <table @wheel="handleWheel" class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Dirección</th>
              <th scope="col">Contenido</th>
              <!--<th scope="col">Contenido en Assembly</th>:key="startIndex + index"-->
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(item, index) in memoriaRam" 
              :class="[(desplazamiento <= index) && (index<=desplazamiento+6)? 'd-table-row' : 'd-none']"
            >
              <td 
                @click="pc = index; errorIns = false;" 
                :class="[pc == index ? 'table-primary' : 'table-ligth']"
                >
                {{ decimalSinSignoAHexadecimal(index)}}
              </td>
              
              <td>
                <input maxlength="4" :value="decimalConSignoAHexadecimal(item)" 
                @input="updateMemory(index,$event.target.value)" type="text"
                @focus="$event.target.select()"
                style="width: 50px; text-align:center"
                >
              </td>
            </tr>
          </tbody>
        </table>
        <div class="row justify-content-between">
          <button @click="desplazamiento++" type="button" class="col btn btn-primary m-2">
            🔽
          </button>
          <button @click="desplazamiento = pc >= 0 && pc < 3 ? pc : pc - 3" type="button" class="col btn btn-primary m-2">
            🟠
          </button>
          <button @click="desplazamiento--" type="button" class="col btn btn-primary m-2">
            🔼
          </button>
        </div>
        <div class="row justify-content-between">
          <button @click="recorrer(false)" :disabled="errorIns" type="button" class="col btn btn-primary m-2">
             ▶️ (en proceso)
          </button>
          <button @click="recorrer(true)" :disabled="errorIns" type="button" class="col btn btn-primary m-2">
            ⏭️ PC: x{{ decimalSinSignoAHexadecimal(pc) }} 
          </button>
        </div>

      </div>

      <div class="col-12 col-sm-6 col-md-1 col-lg-4 col-xl-4 col-xxl-4 animate__animated animate__fadeInRight ">
        <Registros 
          :registros="registros"
          @limpiarRegistros = "limpiarRegistros"
        />
      </div>
    </div>
  </div>
  <footer>Parra Cristian V.R. -- Laboratorio 5 -- Arquitectura de Computadoras</footer>
</template>

<style scoped>
textarea.numbered {
  background: url(http://i.imgur.com/2cOaJ.png);
  background-attachment: local;
  background-repeat: no-repeat;
  padding-left: 45px;
  border-color: #ccc;
  color: black;
  background-position: left calc(0.046%);
  background-size: 39px;
}

.mostrar{
  display:block
}
.ocultar{
  display: none
}
</style>
