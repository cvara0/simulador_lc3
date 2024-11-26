<script setup>
import { ref, reactive, onMounted, computed,onUnmounted, watch } from "vue"
import Registros from "./components/Registros.vue"

const registros = ref([])
const memoriaRam = ref([])
const aBuscar = ref('')
const errores = ref([])
const signo = ref("")
const insTextoHexa = reactive({
  texto:`1265
14A3
1642
5642
5664
96FF
33FF`,
})

const listaInsHexa = ref([])
const inicio = ref("3000")
const desplazamiento = ref(12285)
const pc = ref(12288)

const scrollDelta = ref(0)

onMounted(() => {
  limpiarMemoria()
  limpiarRegistros()
  window.addEventListener('wheel', handleWheel)
});

onUnmounted(() => {
      window.removeEventListener('wheel', handleWheel)
    });


const handleWheel = (event) => {
      scrollDelta.value = event.deltaY
      if(Math.sign(event.deltaY) == 1){
        desplazamiento.value++
      }else{
        desplazamiento.value--
      }
      desplazamiento.value = desplazamiento.value <= 0 ? 0 : desplazamiento.value > 16374 ? 16374 : desplazamiento.value
    };

watch(aBuscar, ()=>{
  if(aBuscar.value)
      desplazamiento.value = hexadecimalADecimalConSigno(aBuscar.value)
  else
    desplazamiento.value = pc.value > 0 && pc.value<3? pc.value -3 : pc.value - 3 
})

///////////////////////////////////////////////////////////////////////////////
const cargar = () => {
  let ini = hexadecimalADecimalConSigno(inicio.value);
  
  const lineas = insTextoHexa.texto
    .trim()
    .split("\n")
    .map((linea) => linea.trim())
 
  errores.value = lineas
    .map((linea, index) => ({ linea: index + 1, valida: /^[0-9A-Fa-f]{4}$/.test(linea) }))
    .filter((r) => !r.valida)

  if (errores.value.length === 0) {
    lineas.forEach((valor, i) => {
      memoriaRam.value[ini + i] = parseInt(valor, 16)
    })
    pc.value = ini
    desplazamiento.value = pc.value > 0 && pc.value<3? pc.value -2 : pc.value - 3
  }
};

///////////////////////////////////////////////////////////////////////////////////
const recorrer = computed(() => {
  
  let insBin = memoriaRam.value[pc.value].toString(2).padStart(16, '0') 
  console.log(insBin);
  
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
      desplazamiento.value = pc.value > 0 && pc.value<3? pc.value -3 : pc.value - 3
    }

  switch (ins.opcode) {
    case "0001": //add
      registros.value[ins.dr] = registros.value[ins.sr1] + (ins.bitImm5 === 0 ? registros.value[ins.sr2] : ins.imm5);
      console.log(registros.value[ins.dr]);
      
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
        desplazamiento.value = pc.value > 0 && pc.value<3? pc.value -3 : pc.value - 3
      break;
    case "1100": //jmp ret
      pc.value = registros.value[ins.baseR];
      desplazamiento.value = pc.value > 0 && pc.value<3? pc.value -3 : pc.value - 3
      break;
    case "0100": //jsr jsrr
      let temp = pc.value;
      pc.value =
        parseInt(insBin.substring(4, 5), 2) === 0
          ? registros.value[ins.baseR]
          : pc.value + ins.PCoffset11;
      registros.value[7] = temp;
      desplazamiento.value = pc.value > 0 && pc.value<3? pc.value -3 : pc.value - 3
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
      break;
  }

  registros.value = registros.value.map((i) => DecimalSignoAHexaGenerator1(i).next().value);
});

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

function decimalASignoHexadecimal(decimal) {
  if (decimal >= 0) {
    // Convertir positivo directamente
    return decimal.toString(16).toUpperCase();
  } else {
    // Calcular complemento a dos para negativos
    const complementoDos = (1 << 16) + decimal;
    return complementoDos.toString(16).toUpperCase();
  }
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

const limpiarMemoria = () => {
  memoriaRam.value = new Int32Array(16384)
}
const limpiarRegistros = () => {
  registros.value = new Array(8).fill("0000")
}

const reiniciarPc = computed(()=>{
  pc.value = hexadecimalADecimalConSigno(inicio.value)
  desplazamiento.value = pc.value < 3 ? pc.value :pc.value - 3
})

const indiceRecorrer = computed(() => decimalASignoHexadecimal(pc.value).padStart(4, "0") ) 

////////////////////////////////////////////////////////////////////////////////
function* DecimalAHexaGenerator(array) {
  for (const decimal of array) {
    yield decimal.toString(16).toUpperCase().padStart(4, '0'); 
  }
}

  function* DecimalAHexaGenerator1(decimal) {
    yield decimal.toString(16).padStart(4, '0')
}

function* DecimalSignoAHexaGenerator1(decimal) {
  if (decimal < 0) {
    const hexValue = (Math.pow(2, 16) + decimal).toString(16).toUpperCase();
    yield hexValue;
  } else {
    const hexValue = decimal.toString(16).toUpperCase().padStart(4, '0');
    yield hexValue;
  }
}

///////////////////////////////////////////////////////////////////////



</script>

<template>
  <h5>Laboratorio 5</h5>
  <h1>Simulador de ISA de la LC3</h1>
  <h5>(Basado en el laboratorio 2)</h5>
  <div class="container mt-5">
    <div class="row">
      <div class="col-4">
        <h4 class="mt-4">Instrucciones en Hexadecimal</h4>

        <form @submit.prevent="cargar" class="form-floating d-grid gap-2">
          <div class="input-group input-group">
            <span class="input-group-text">Iniciar en X: </span>
            <input v-model="inicio" type="text" class="form-control" maxlength="4"/>
          </div>
          <button type="submit" class="btn btn-primary">Cargar a memoria ⤴</button>
          <textarea
            v-model="insTextoHexa.texto"
            class="numbered"
            style="height: 400px"
          > </textarea>
        </form>

        <div v-if="errores.length > 0" class="mt-2">
          <div class="alert alert-danger" role="alert">Errores de carga:</div>
          <ul v-for="item in errores" class="list-group">
            <li class="list-group-item">En linea {{ item.linea }}</li>
          </ul>
        </div>
        <div v-else class="mt-2">
          <div class="alert alert-success" role="alert">👍</div>
        </div>
      </div>

      <div class="col-4">
        <h4>Memoria RAM {{ memoriaRam.length }}</h4>
        <div class="row justify-content-between">
          <button @click="limpiarMemoria" type="button" class="col btn btn-danger m-2">
            Limpiar memoria
          </button>
          <button @click="reiniciarPc" type="button" class="col btn btn-danger m-2">
            Reiniciar PC
          </button>
          <button @click="recorrer" type="button" class="col btn btn-primary m-2">
            Recorrer PC: X{{ indiceRecorrer }} ⤵
          </button>
    
          <div class="input-group">
            <span class="input-group-text" id="inputGroup-sizing-default"> 🔍Buscar dirección: x</span>
            <input v-model="aBuscar" type="text" maxlength="4" class="form-control" aria-label="Sizing example input" aria-describedby="inputGroup-sizing-default">
          </div>
          <button @click="desplazamiento++" type="button" class="col btn btn-primary m-2">
            ↓
          </button>
          <button @click="desplazamiento--" type="button" class="col btn btn-primary m-2">
            ↑
          </button>
        </div>

        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Dirección</th>
              <th scope="col">Contenido</th>
              <!--<th scope="col">Contenido en Assembly</th>-->
            </tr>
          </thead>
          <tbody>
            <tr
              @click="pc = index"
              v-for="(item, index) in DecimalAHexaGenerator(memoriaRam)"
              :class="[pc == index ? 'table-primary' : 'table-ligth']"
              v-show="(desplazamiento <= index) && (index<=desplazamiento+7)"
            >
              <td>x{{ DecimalAHexaGenerator1(index).next().value }}</td>
              <td>x{{ item }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="col-2">
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
</style>
