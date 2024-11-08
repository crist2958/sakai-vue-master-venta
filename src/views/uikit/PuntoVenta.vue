<template>
    <div class="p-grid">
        <Toast />
        <div class="p-col-12">
            <div class="card">
                <Panel header="PUNTO DE VENTA (POS)" style="height: 100%">
                    <Toolbar class="p-mb-4">
                        <template v-slot:start>
                            <InputText type="text" size="40" placeholder="Nombre del producto..." v-model="productoItem.nomProducto" />
                            <InputText class="ml-8" type="number" placeholder="Cant" v-model="productoItem.cantidad" />
                            <InputText class="ml-8" type="number" placeholder="$ Precio U." v-model="productoItem.precioUnitario" />
                        </template>
                        <template v-slot:end>
                            <Button label="Registrar" icon="pi pi-plus" class="p-button-success p-mr-2" @click="registrarCompra()" />
                        </template>
                    </Toolbar>
                    <br />

                    <DataTable :value="tablaCompras" class="p-datatable-gridlines" dataKey="cns" :rows="5" paginator
                        paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                        :rowsPerPageOptions="[5,10,25]"
                        currentPageReportTemplate="Visualizando {first} a {last} de {totalRecords} productos"
                        responsiveLayout="scroll">
                        
                        <Column field="cns" header="Cns" :sortable="true" style="width:50px"></Column>
                        <Column field="nomProducto" header="Nombre del Producto" style="width:370px"></Column>
                        <Column field="precioUnitario" header="Precio U." dataType="numeric" style="width:80px">
                            <template #body="slotProps">
                                {{ formatoMoneda(slotProps.data.precioUnitario) }}
                            </template>
                        </Column>
                        <Column field="cantidad" header="Cant." style="width:60px;text-align:center"></Column>
                        <Column field="precioParcial" header="Precio P." style="width:80px">
                            <template #body="slotProps">
                                {{ formatoMoneda(slotProps.data.precioParcial) }}
                            </template>
                        </Column>
                        <Column style="width:140px">
                            <template #header>Acciones</template>
                            <template #body="slotProps">
                                <Button icon="pi pi-pencil" class="p-button-success p-mr-2 p-mb-1" @click="confirmarEditarProducto(slotProps.data)" />
                                <Button icon="pi pi-trash" class="p-button-danger p-mb-1" @click="confirmarEliminarProducto(slotProps.data)" />
                            </template>
                        </Column>
                    </DataTable>

                    <div class="totales" style="margin-top: 1rem;">
                        <div style="display: inline-block; margin-right: 1rem;">
                            <strong>Subtotal:</strong>
                            <InputText type="text" :value="formatoMoneda(subtotal)" disabled />
                        </div>
                        <div style="display: inline-block; margin-right: 1rem;">
                            <strong>IVA (16%):</strong>
                            <InputText type="text" :value="formatoMoneda(iva)" disabled />
                        </div>
                        <div style="display: inline-block;">
                            <strong>Total:</strong>
                            <InputText type="text" :value="formatoMoneda(totalTotal)" disabled />
                        </div>
                    </div>
                </Panel>
            </div>
        </div>
    </div>

    <Dialog header="Confirmar Eliminación" v-model:visible="visibleDelete" modal>
        <p>¿Está seguro de que desea eliminar el producto "{{ productoSeleccionado.nomProducto }}"?</p>
        <div class="p-dialog-footer">
            <Button label="No" icon="pi pi-times" class="p-button-text" @click="visibleDelete = false" />
            <Button label="Sí" icon="pi pi-check" class="p-button-text" @click="deleteProducto()" />
        </div>
    </Dialog>

    <Dialog header="Confirmar Edición" v-model:visible="visibleUpdate" modal>
        <div>
            <p>Editar el producto "{{ productoSeleccionado.nomProducto }}"</p>
            <InputText type="text" placeholder="Nombre del producto..." v-model="productoItem.nomProducto" />
            <InputText type="number" placeholder="Cantidad" v-model="productoItem.cantidad" />
            <InputText type="number" placeholder="$ Precio U." v-model="productoItem.precioUnitario" />
        </div>
        <div class="p-dialog-footer">
            <Button label="Cancelar" icon="pi pi-times" class="p-button-text" @click="visibleUpdate = false" />
            <Button label="Actualizar" icon="pi pi-check" class="p-button-text" @click="updateProducto()" />
        </div>
    </Dialog>
</template>
<script>
import { useToast } from 'primevue/usetoast';
import { computed, ref } from 'vue';

export default {
    setup() {
        const toast = useToast();
        const visibleUpdate = ref(false);
        const visibleDelete = ref(false);
        const productoSeleccionado = ref(null);

        const productoItem = ref({
            nomProducto: '',
            cantidad: '',
            precioUnitario: ''
        });

        const tablaCompras = ref([ 
            { cns: 1, nomProducto: 'Impresora LaserJet Color', cantidad: 2, precioUnitario: 5200.00, precioParcial: 10400.00 }, 
            { cns: 2, nomProducto: 'Monitor LED 31 plg.', cantidad: 3, precioUnitario: 1700.00, precioParcial: 5100.00 } 
        ]);

        const registrarCompra = () => {
            if (productoItem.value.nomProducto && productoItem.value.cantidad && productoItem.value.precioUnitario) {
                const nuevoProducto = {
                    cns: tablaCompras.value.length + 1,
                    nomProducto: productoItem.value.nomProducto,
                    cantidad: parseFloat(productoItem.value.cantidad),
                    precioUnitario: parseFloat(productoItem.value.precioUnitario),
                    precioParcial: parseFloat(productoItem.value.cantidad) * parseFloat(productoItem.value.precioUnitario)
                };
                tablaCompras.value.push(nuevoProducto);

                toast.add({ severity: 'success', summary: 'Confirmación', detail: 'Nueva compra registrada', life: 3000 });

                productoItem.value.nomProducto = '';
                productoItem.value.cantidad = '';
                productoItem.value.precioUnitario = '';
            } else {
                toast.add({ severity: 'warn', summary: 'Advertencia', detail: 'Todos los campos son obligatorios', life: 3000 });
            }
        };

        const confirmarEditarProducto = (producto) => {
            productoSeleccionado.value = producto;
            productoItem.value = { ...producto }; // Copiar los datos del producto seleccionado
            visibleUpdate.value = true;
        };

        const updateProducto = () => {
            const index = tablaCompras.value.findIndex(producto => producto.cns === productoSeleccionado.value.cns);
            if (index !== -1) {
                tablaCompras.value[index] = {
                    ...productoItem.value,
                    cns: productoSeleccionado.value.cns, // Mantener el mismo número de producto
                    precioParcial: productoItem.value.cantidad * productoItem.value.precioUnitario // Recalcular el precio parcial
                };
            }
            visibleUpdate.value = false;
            toast.add({ severity: 'info', summary: 'Producto actualizado', detail: 'El producto ha sido actualizado', life: 3000 });
        };

        const confirmarEliminarProducto = (producto) => {
            productoSeleccionado.value = producto;
            visibleDelete.value = true;
        };

        const deleteProducto = () => {
            tablaCompras.value = tablaCompras.value.filter(producto => producto.cns !== productoSeleccionado.value.cns);
            visibleDelete.value = false;
            toast.add({ severity: 'error', summary: 'Producto eliminado', detail: 'El producto ha sido eliminado', life: 3000 });
        };

        const formatoMoneda = (value) => {
            if (value) return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
            return '';
        };

        const subtotal = computed(() => {
            return tablaCompras.value.reduce((acc, item) => acc + item.precioParcial, 0);
        });

        const iva = computed(() => {
            return subtotal.value * 0.16;
        });

        const totalTotal = computed(() => {
            return subtotal.value + iva.value;
        });

        return {
            registrarCompra,
            confirmarEditarProducto,
            updateProducto,
            confirmarEliminarProducto,
            deleteProducto,
            formatoMoneda,
            productoItem,
            tablaCompras,
            visibleUpdate,
            visibleDelete,
            productoSeleccionado,
            subtotal,
            iva,
            totalTotal,
            toast,
        };
    }
};
</script>
