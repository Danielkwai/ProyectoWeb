<script setup>
import bwipjs from 'bwip-js';
import { nextTick, ref } from 'vue';

import Button from 'primevue/button';
import Calendar from 'primevue/calendar';
import InputText from 'primevue/inputtext';

const form = ref({
    nombre: '',
    paterno: '',
    materno: '',
    fecha: null
});

const rfc = ref('');
const rfcError = ref('');
const canvasRef = ref(null);

function validarRFC() {
    const val = rfc.value?.toUpperCase() || '';

    const regex = /^[A-ZÑ&]{4}\d{6}[A-Z0-9]{3}$/;

    if (!regex.test(val)) {
        rfcError.value = 'RFC inválido';
        return false;
    }

    rfcError.value = '';
    return true;
}

function onSubmit() {
    if (!validarRFC()) return;

    generarBarcodeManual();
}

async function generarBarcodeManual() {
    await nextTick();

    if (!canvasRef.value) return;

    try {
        bwipjs.toCanvas(canvasRef.value, {
            bcid: 'code128',
            text: rfc.value,
            scale: 2,
            height: 8,
            includetext: false,
            backgroundcolor: 'FFFFFF'
        });

        canvasRef.value.style.width = '100%';
        canvasRef.value.style.height = 'auto';
    } catch (err) {
        console.error(err);
    }
}
</script>

<template>
    <Fluid>
        <div class="flex flex-col md:flex-row gap-8">
            <!-- FORM -->
            <div class="w-full md:w-1/2">
                <div class="card flex flex-col gap-4">
                    <div class="font-semibold text-xl">GENERAR EL RFC DE UNA PERSONA</div>

                    <div class="flex flex-col gap-2">
                        <label>Nombre(s)</label>
                        <InputText v-model="form.nombre" />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label>Apellido Paterno</label>
                        <InputText v-model="form.paterno" />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label>Apellido Materno</label>
                        <InputText v-model="form.materno" />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label>Fecha Nacimiento</label>
                        <Calendar v-model="form.fecha" showIcon />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label>RFC (13 caracteres)</label>
                        <InputText v-model="rfc" maxlength="13" @input="rfc = rfc.toUpperCase()" placeholder="Ej: ABCD010203XYZ" />
                        <small v-if="rfcError" style="color: red">{{ rfcError }}</small>
                    </div>

                    <Button label="VALIDAR Y GENERAR" class="p-button-success w-full mt-2" @click="onSubmit" />
                </div>
            </div>

            <!-- TARJETA -->
            <div class="w-full md:w-1/2 flex justify-center">
                <div class="rfc-card w-full max-w-sm">
                    <!-- HEADER -->
                    <div class="rfc-header">
                        <strong>UNACH</strong>
                        <small>LIDTS</small>
                    </div>

                    <!-- BODY -->
                    <div class="rfc-body text-center">
                        <i class="pi pi-user text-4xl mb-3"></i>

                        <!-- NOMBRE -->
                        <div class="name">
                            {{ (form.nombre + ' ' + form.paterno + ' ' + form.materno).toUpperCase() }}
                        </div>

                        <div class="role">Empleado</div>

                        <!-- RFC -->
                        <div class="rfc-text mt-2">
                            {{ rfc }}
                        </div>

                        <!-- BARCODE CON CONTENEDOR -->
                        <div v-if="rfc" class="barcode-container">
                            <canvas ref="canvasRef"></canvas>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </Fluid>
</template>

<style scoped>
.rfc-card {
    border-radius: 20px;
    overflow: hidden;
    background: linear-gradient(180deg, #0f3c5c, #2d8cf0);
    color: white;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}

.rfc-header {
    text-align: center;
    padding: 1.2rem;
    background: rgba(0, 0, 0, 0.25);
    display: flex;
    flex-direction: column;
    font-size: 1.1rem;
}

.rfc-body {
    padding: 2rem 1.5rem;
}

.name {
    font-weight: 700;
    font-size: 1.1rem;
    margin-top: 0.5rem;
}

.role {
    opacity: 0.8;
    font-size: 0.9rem;
    margin-top: 0.3rem;
}

.rfc-text {
    font-weight: 600;
    letter-spacing: 1px;
    margin-top: 0.5rem;
}

.barcode-container {
    margin-top: 1.5rem;
    background: white;
    padding: 0.5rem;
    border-radius: 8px;
    width: 100%;
    max-width: 260px; /* 🔥 límite real */
    margin-left: auto;
    margin-right: auto;
    overflow: hidden; /* 🔥 evita que se salga */
}

.barcode-container canvas {
    display: block;
    width: 100%;
    height: auto;
}
</style>
