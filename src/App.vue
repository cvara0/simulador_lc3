<script setup>
import { ref, reactive, onMounted, computed,onUnmounted, watch } from "vue"
import Registros from "./components/Registros.vue"
import { useVirtualList } from "@vueuse/core"
//subir archivo cargar en textarea drag and drop, ver lenguaje assembly, agregar nzps


const registros = ref([])
const memoriaRam = ref(new Array(65536).fill("0000"))

const aBuscarIndex = ref('')
const aBuscarIns = ref('')
const errores = ref([])
const errorIns = ref(false)
const cantIns = ref(0)
const signo = ref("")
const msjHalt = ref("")


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
const pc = ref(12288)

onMounted(() => {
  limpiarMemoria()
  limpiarRegistros()
  scrollTo(pc.value-7)
  //window.addEventListener('wheel', handleWheel)
})

const limpiarMemoria = () => {
  errorIns.value = false
  memoriaRam.value = new Array(65536).fill("0000")
  pc.value = hexadecimalADecimalConSigno(inicio.value)
  scrollTo(pc.value-7)
}
const limpiarRegistros = () => {
  registros.value = new Array(8).fill("0000")
}

const { list, containerProps, wrapperProps, scrollTo } = useVirtualList(memoriaRam.value, {
      itemHeight: 34, // Ajusta la altura de cada fila en píxeles
    });

/*  const handleWheel = (event) => {
      scrollDelta.value = event.deltaY
      if(Math.sign(event.deltaY) == 1){
        desplazamiento.value++
      }else{
        desplazamiento.value--
      }
      desplazamiento.value = desplazamiento.value <= 0 ? 0 : desplazamiento.value > 35536? 35536 : desplazamiento.value
    
    } */ 

watch(aBuscarIndex, ()=>{
  if(aBuscarIndex.value)
    scrollTo(hexadecimalADecimalSinSigno(aBuscarIndex.value)) 
  else
    scrollTo(pc.value - 7)
}) 

watch(aBuscarIns, ()=>{
  if(aBuscarIns.value){
    scrollTo(memoriaRam.value.findIndex((item) => item?.toString().toLowerCase().includes(aBuscarIns.value.toLowerCase()))) 
  }
    else
    scrollTo(pc.value - 7)
}) 

///////////////////////////////////////////////////////////////////////////////
function cargar(){
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
      memoriaRam.value[ini + i] = valor
    })
    pc.value = ini
    cantIns.value = lineas.length
    scrollTo(pc.value-7)
  }
}

///////////////////////////////////////////////////////////////////////////////////
function recorrer(esRecorrer){
  if (esHexadecimal(memoriaRam.value[pc.value])){
    if(esRecorrer){
        procesarIns()
      }
    else{
      while(!errorIns.value) {
        procesarIns()
      }
    }
  } 
  else{
        errorIns.value = true
        msjHalt.value = "Instrucción desconocida: "+ memoriaRam.value[pc.value]
      }
} 

////////////////////////////////////////////////////////////////////////////////

const procesarIns = () => {
  if (memoriaRam.value[pc.value] === "0000") {
    errorIns.value = true
    msjHalt.value= "Ejecución finalizada"
  } else {
    let insBin = hexadecimalABinario16(memoriaRam.value[pc.value])
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

    if (pc.value > 35356) 
      pc.value = 35356
    else 
      {
        pc.value++
        scrollTo(pc.value-7)
      }
    switch(ins.opcode) {
      case "0001": //add
        registros.value[ins.dr] = registros.value[ins.sr1] + (ins.bitImm5 === 0 ? registros.value[ins.sr2] : ins.imm5)
        signo.value = Math.sign(registros.value[ins.dr]);
        break
      case "0101": //and
        registros.value[ins.dr] = registros.value[ins.sr1] & (ins.bitImm5 === 0 ? registros.value[ins.sr2] : ins.imm5)
        signo.value = Math.sign(registros.value[ins.dr]);
        break
      case "1001": //not
        registros.value[ins.dr] = ~registros.value[ins.sr1]
        signo.value = Math.sign(registros.value[ins.dr])
        break
      case "0000": //br
        if ((ins.n === "1" && signo.value === -1) || (ins.z === "1" && signo.value === 0) || (ins.p === "1" && signo.value === 1))
          pc.value = pc.value + ins.PCoffset9
          scrollTo(pc.value-7)
        break;
      case "1100": //jmp ret
        pc.value = registros.value[ins.baseR]
        scrollTo(pc.value-7)
        break;
      case "0100": //jsr jsrr
        let temp = pc.value;
        pc.value =
          parseInt(insBin.substring(4, 5), 2) === 0 ? registros.value[ins.baseR] : pc.value + ins.PCoffset11
          registros.value[7] = temp
          scrollTo(pc.value-7)
        break;
      case "0010": //ld
        registros.value[ins.dr] = hexadecimalADecimalConSigno(memoriaRam.value[pc.value + ins.PCoffset9])
        break;
      case "1010": //ldi
        registros.value[ins.dr] = hexadecimalADecimalConSigno(memoriaRam.value[memoriaRam.value[pc.value + ins.PCoffset9]])
        break;
      case "0110": //ldr
        registros.value[ins.dr] = hexadecimalADecimalConSigno(memoriaRam.value[registros.value[ins.baseR] + ins.offset6])
        break;
      case "1110": //lea
        registros.value[ins.dr] = pc.value + ins.PCoffset9;
        break;
      case "0011": //st
        memoriaRam.value[pc.value + ins.PCoffset9] = decimalConSignoAHexadecimal(registros.value[ins.sr])
        break;
      case "1011": //sti
        memoriaRam.value[hexadecimalADecimalConSigno(memoriaRam.value[pc.value + ins.PCoffset9])] = decimalConSignoAHexadecimal(registros.value[ins.sr])
        break;
      case "0111": //str
        memoriaRam.value[registros.value[ins.baseR] + ins.offset6] = decimalConSignoAHexadecimal(registros.value[ins.sr])
        break;
      default:
        errorIns.value = true
        msjHalt.value = "Instrucción desconocida: "+memoriaRam.value[pc.value-1]
        pc.value--
        break;
    }
    registros.value = registros.value.map((i) => decimalConSignoAHexadecimal(i))
  }
}

///////////////////////////////////////////////////////////////////////
const esHexadecimal= (valor) => /^[0-9a-fA-F]+$/.test(valor)

const hexadecimalABinario16 = (hex) => parseInt(hex, 16).toString(2).padStart(16, '0')

const binarioC2ADecimal = (binario) => (binario[0] === "1")? -((~parseInt(binario, 2) + 1) & ((1 << binario.length) - 1)):parseInt(binario, 2)
    

/* function decimalABinarioC2(decimal){
  const mask = (1 << 16) - 1;
  return (decimal & mask).toString(2).padStart(16, '0');
}
 */
const decimalConSignoAHexadecimal = (sdecimal) =>(sdecimal >= 0)?sdecimal.toString(16).padStart(4, '0').toUpperCase():(Math.pow(2, 16) + sdecimal).toString(16).padStart(4, '0').toUpperCase()
 
const decimalSinSignoAHexadecimal = (udecimal) => udecimal.toString(16).toUpperCase().padStart(4, '0')

const hexadecimalADecimalSinSigno = (hex) => parseInt(hex, 16)

const hexadecimalADecimalConSigno = (hex) => (parseInt(hex, 16) >= Math.pow(2, 16 - 1))?parseInt(hex, 16) - Math.pow(2, 16):parseInt(hex, 16)
  
const reiniciarPc = ()=>{
  errorIns.value = false
  pc.value = hexadecimalADecimalConSigno(inicio.value)
  scrollTo(pc.value-7)
 }

///////////////////////////////////////////////////////////////////////////////

const contarIns = computed(() => (insTextoHexa.texto.trim() === '') ? 0 : insTextoHexa.texto.split('\n').filter(line => line.trim() !== '').length)
  
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
        <div class="mb-3">
          <label for="formFileSm" class="form-label">Subir archivo .txt con las instrucciones (en proceso)</label>
          <input class="form-control form-control-sm" id="formFileSm" type="file">
        </div>
        <form @submit.prevent="cargar" class="form-floating d-grid gap-2">

          <div class="form-floating mb-2">
            <input v-model="inicio" maxlength="4" @focus="$event.target.select()" type="text" class="form-control" id="floatingInput" placeholder="name@example.com">
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
          <div class="alert alert-danger" role="alert">Errores de sintáxis:</div>
          <ul v-for="item in errores" class="list-group">
            <li class="list-group-item">En linea {{ item.linea }}</li>
          </ul>
        </div>
        <div v-else-if="memoriaRam.some((i) => i !== '0000')" class="mt-2">
          <div class="alert alert-success" role="alert">Carga OK👍</div>
        </div>
      </div>


      <div class="col-12 col-sm-6 col-md-6 col-lg-4 col-xl-4	col-xxl-4  animate__animated animate__fadeInUp">
        <h4>Memoria RAM {{ memoriaRam.length }} </h4>
        <div class="row justify-content-between">
          <button @click="limpiarMemoria" type="button" class="col btn btn-danger m-2">
            Limpiar memoria
          </button>
          <button @click="reiniciarPc()" type="button" class="col btn btn-danger m-2">
            Reiniciar PC
          </button>
          <div class="input-group mb-2">
            <input v-model="aBuscarIndex" :disabled="aBuscarIns !== ''" @focus="$event.target.select()" maxlength="4" type="text" class="form-control" placeholder="🔍Dirección" aria-label="Dirección">
            <input v-model="aBuscarIns" :disabled="aBuscarIndex !== ''" @focus="$event.target.select()" maxlength="4" type="text" class="form-control" placeholder="🔍Instrucción" aria-label="Instrucción">
          </div>
        </div>
 
        <div v-bind="containerProps" class="virtual-container table table-hover table-light"> 
            <div v-bind="wrapperProps"class="virtual-wrapper container text-center">
          
              <div
                v-for="item in list"
                :key="item.index"
                class="virtual-item  row row-cols-2"
              >
                <div
                  @click="pc = item.index; errorIns = false;" 
                  :class="[pc == item.index ? 'table-primary' : '']"
                  class="col text-center"
                  >{{ decimalSinSignoAHexadecimal(item.index) }}
                </div>
                
                  <input
                    type="text"
                    v-model="memoriaRam[item.index]"
                    @focus="$event.target.select()"
                    maxlength="4"
                    class="col text-center"
                  />
              </div>
            </div>
          </div>
            
          <div v-show="errorIns" class="alert alert-danger" role="alert">
          ⛔  {{msjHalt}}  ⛔
          </div>
      
            <!-- desplazamiento++   desplazamiento = pc >= 0 && pc < 3 ? pc : pc - 3  desplazamiento---->
        
     
        <div class="row justify-content-between">
          <button @click="recorrer(false)" :disabled="errorIns" type="button" class="col btn btn-primary m-2">
             ▶️
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



    
  <div class="mt-4" style="text-align:justify;">
    <h5>Consideraciones</h5>
    <p >
    Realizar este laboratorio en Vue.js tiene tanto ventajas como desventajas:
    <h6>Ventajas:</h6>
    <ol>
      <li><b>Administración de componentes y comunicación entre ellos:</b>  Vue facilita la interacción entre diferentes elementos de la aplicación, como entradas de datos (inputs) y operaciones, permitiendo que los cambios se reflejen casi instantáneamente al modificar una variable.</li>
      <li><b>Interfaz de usuario amigable y flexible:</b>  El framework permite crear interfaces más atractivas y dinámicas, con alta capacidad de personalización en cuanto a la visualización y manipulación de datos.</li>
      <li><b>Velocidad de desarrollo:</b>  Vue ofrece herramientas como clases y métodos predefinidos que simplifican la creación de componentes y la vinculación entre entradas y procesamiento de datos, acelerando significativamente el proceso de programación.</li>
      <li><b>Uso de Bootstrap:</b> Es un framework web que facilita crear sitios responsivos con componentes predefinidos, grillas y utilidades CSS, agilizando el diseño y desarrollo de interfaces atractivas.</li>
    </ol>
    <h6>Desventajas:</h6>
    <ol>
      <li><b>Consumo de recursos (resuelto):</b> Simular una memoria precargada con valores resulta engorroso y costoso en términos de rendimiento. Con mi experiencia actual, no fue posible simular una memoria de 65.535 posiciones debido al alto consumo de recursos. Como solución temporal, se redujo a 32.768 posiciones, aunque el consumo sigue siendo elevado. En el futuro, se explorarán alternativas para mejorar la eficiencia y reducir el impacto en el rendimiento.</li>
    </ol>
    <h6>Conclusión:</h6>
    <p>En todo momento, la herramienta de IA generativa ha sido de gran ayuda, permitiendo reducir significativamente los tiempos de programación. Su aporte ha sido especialmente valioso en la creación de métodos específicos, la clarificación de conceptos y la simplificación del código, logrando así un desarrollo más eficiente.
      El desarrollo de software es un proceso iterativo e incremental, en el que siempre surgen errores o áreas de mejora que deben resolverse. El objetivo principal es minimizar la aparición de problemas en el uso normal de la aplicación. Esta aplicación no está exenta de errores o detalles que, con una exploración más profunda, los usuarios podrán descubrir.</p>
    </p>
    <h6>Instrucciones de prueba:</h6>
</div>

    <ul class="list-group list-group-flush"  style="text-align:justify;">
      <li class="list-group-item">1265 ::: ADD R1, R1, #5 ::: R1 = 0005 </li>
      <li class="list-group-item">14A3 ::: ADD R2, R2, #3 ::: R2 = 0003 </li>
      <li class="list-group-item">1642 ::: ADD R3, R1, R2 ::: R3 = 0008 </li>
      <li class="list-group-item">5642 ::: AND R3, R1, R2 ::: R3 = 0001 </li>
      <li class="list-group-item">5664 ::: AND R3, R1, #4 ::: R3 = 0004 </li>
      <li class="list-group-item">96FF ::: NOT R3, R3 ::: R3 = FFFB </li>
      <li class="list-group-item">33FF ::: ST R1, #0 ::: M[3006] = 0005 </li>
      <li class="list-group-item">29FE ::: LD R4, #-2 ::: R4 = 0005 </li>
      <li class="list-group-item">6A84 ::: LDR R5, R2, #4 ::: R5 = M[0007] </li>
      <li class="list-group-item">7643 ::: STR R3, R1, #3 ::: M[0008] = FFFB </li>
      <li class="list-group-item">1CA0 ::: ADD R6, R2, #0 ::: R6 = R2 </li>
      <li class="list-group-item">1DBF ::: ADD R6, R6, #-1 ::: R6 = R6 - 1 </li>
      <li class="list-group-item">07FE ::: BRzp #-2 ::: Vuelve a la instrucción anterior, R6 = R6 - 1, hasta que R6 sea negativo </li>
      <li class="list-group-item">1CA0 ::: ADD R6, R2, #0 ::: R6 = R2 </li>
      <li class="list-group-item">1DBF ::: ADD R6, R6, #-1 ::: R6 = R6 - 1  </li>
      <li class="list-group-item">03FE ::: BRp #-2 ::: Vuelve a la instrucción anterior, R6 = R6 - 1, hasta que R6 sea cero o negativo</li>
      <li class="list-group-item">1CE0 ::: ADD R6, R3, #0 ::: R6 = R3  </li>
      <li class="list-group-item">1DA1 ::: ADD R6, R6, #1 ::: R6 = R6 + 1 </li>
      <li class="list-group-item">0DFE ::: BRnz #-2 ::: Vuelve a la instrucción anterior, R6 = R6 + 1, hasta que R6 sea positivo</li>
      <li class="list-group-item">1CE0 ::: ADD R6, R3, #0 ::: R6 = R3 </li>
      <li class="list-group-item">1DA1 ::: ADD R6, R6, #1 ::: R6 = R6 + 1 </li>
      <li class="list-group-item">09FE ::: BRn #-2 ::: Vuelve a la instrucción anterior, R6 = R6 + 1, hasta que R6 sea cero o positivo </li>
      <li class="list-group-item">C040 ::: JMP R1 ::: PC = R1 </li>
    </ul>
      

  </div>
  <footer>Parra Cristian V.R. -- Laboratorio 5 -- Arquitectura de Computadoras</footer>
</template>

<style scoped>
.recycle-scroller {
  max-height: 400px; /* Altura máxima del área visible */
  overflow-y: auto;/* Habilitar desplazamiento vertical */
}
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


.virtual-container {
  height: 500px; /* Ajusta la altura visible */
  overflow-y: auto; /* Desplazamiento vertical */
  
}

.virtual-wrapper {
  position: relative;
}

.virtual-item {
  height: 34px; /* Debe coincidir con `itemHeight` */
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 10px;
  border-bottom: 1px solid #eee;
}

.memory-input {
  width: 60px;
  height: 32px;
  text-align: center;
}



</style>
