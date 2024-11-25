<script setup>

    const props = defineProps({
        desplazamiento:{
            type: Number,
            required: true
        },
        inicio:{
            type: Number,
            required: true
        },
        pc:{
            type: Number,
            required: true
        },
        memoriaRam:{
            type: Int32Array,
            required: true
        }
    })
    defineEmits([
        'limpiarMemoria',
        'hexadecimalADecimalConSigno',
        'recorrer'
    ])

function* DecimalAHexaGenerator(array) {
  for (const decimal of array) {
    yield decimal.toString(16).padStart(4, '0')
  }
}
function* DecimalAHexaGenerator1(decimal) {
    yield decimal.toString(16).padStart(4, '0')
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

</script>

<template>
    <h4>Memoria RAM {{ memoriaRam.length }}</h4>

    <div class="row justify-content-between">
    <button @click="limpiarMemoria" type="button" class="col btn btn-danger m-2">
        Limpiar memoria
    </button>
    <button @click="pc = hexadecimalADecimalConSigno(inicio); desplazamiento = pc - 5 " type="button" class="col btn btn-danger m-2">
        Reiniciar PC
    </button>
    <button @click="recorrer" type="button" class="col btn btn-primary m-2">
        Recorrer PC: X{{ decimalASignoHexadecimal(pc).padStart(4, "0") }} ⤵
    </button>
    <div class="form-floating">
        <input class="form-control" placeholder="Leave a comment here" id="floatingTextarea">
        <label for="floatingTextarea" > 🔍 X (en desarrollo)</label>
    </div>
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
        v-show="(desplazamiento <= index) && (index<=desplazamiento+9)"
        >
            <td>x{{ DecimalAHexaGenerator1(index).next().value }}</td>
            <td>x{{ item }}</td>
        </tr>
    </tbody>
    </table>
</template>