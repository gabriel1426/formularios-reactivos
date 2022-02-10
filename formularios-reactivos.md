# Formularios reactivos

Los Formularios Reactivos nos proveen de una forma de manejar las entradas de datos del usuario cuyos valores cambian en el tiempo.

Cada cambio que ocurre en el formulario devuelve un nuevo estado, lo que ayuda a mantener la integridad del modelo entre cada cambio. Los formularios reactivos están basados en flujos de datos de tipo Observable, donde cada entrada y cada valor toman la forma de un flujo de datos que puede ser accedido de manera asíncrona.

Los Formularios Reactivos son mas escalables, reusables y fáciles de probar. Cada Elemento de la vista está directamente enlazado al modelo mediante una instancia de FormControl. Las actualizaciones de la vista al modelo y del modelo a la vista son síncronas y no dependen de la representación en la Interfaz de usuario del cliente.

## Form Builder

El FormBuilder es un servicio del que han de depender los componentes que quieran desacoplar el modelo de la vista. Se usa para construir un formulario creando un FormGroup, (un grupo de controles) que realiza un seguimiento del valor y estado de cambio y validez de los datos.

Para poder usarlo tenemos que importar el módulo de Angular en el que viene declarado, el ReactiveFormModule.

#### transactions.module.ts
```typescript

import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  declarations: [RegisterComponent],
  imports: [
    CommonModule,
    TransactionRoutingModule,
    ReactiveFormsModule
  ]
})
export class TransactionsModule { }
```

Veamos un ejemplo de su declaración.

#### bhd-accounts-transaction.component.ts
```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

export class BhdAccountsTransactionComponent implements OnInit {
  public formGroup: FormGroup;

  constructor( private formBuilder: FormBuilder ) { }

  public ngOnInit() {
    this.buildForm();
  }

  private buildForm(){
    this.formGroup = this.formBuilder.group({});
  }
}
```

## Form control

El formulario se define como un grupo de controles. Cada control tendrá un nombre y una configuración. Esa definición permite establecer un valor inicial al control.

#### bhd-accounts-transaction.component.ts
```typescript
private buildForm() {
  this.accountsForm = this.fb.group({
    amount: [],
    currency: [],
    description: [],
    isToday: [true],
    date: [PrivateUtilsService.getCurrentDate()]
  });
}
```

Como ves, es fácil asignar valores por defecto. Incluso es un buen momento para modificar o transformar datos previos para ajustarlos a cómo los verá el usuario; sin necesidad de cambiar los datos de base.

## Form view

Mientras tanto en la vista html… Este trabajo previo y extra que tienes que hacer en el controlador se recompensa con una mayor limpieza en la vista. Lo único necesario será asignar por nombre el elemento html con el control typescript que lo gestionará.

Para ello usaremos dos directivas que vienen dentro del módulo reactivo son **[formGroup]="objetoFormulario"** para el formulario en su conjunto, y **formControlName="nombreDelControl"** para cada control.

#### register.component.html
```html
<form [formGroup]="accountsForm">
  <label for="amount">amount</label>
  <input name="amount"
        formControlName="amount"
        type="number" />
  <label for="currency">Currency</label>
  <input name="currency"
        formControlName="currency"
        type="text" />
  <label for="description">Description</label>
  <input name="description"
        formControlName="description"
        type="text" />
  <label for="isToday">IsToday</label>
  <input name="isToday"
        formControlName="isToday"
        type="radio" />
  <label for="date">Date</label>
  <input name="date"
        formControlName="date"
        type="date" />
</form>
```

## Validadores predefinidos y personalizados

La validación es una pieza clave de la entrada de datos en cualquier aplicación. Como veremos, tenemos la posibilidad de escribir nuestras propias validaciones para resolver diferentes casos y de igual forma se pueden usar las validaciones que vienen predefinidas como funciones en el objeto Validators de framework.

#### bhd-accounts-transaction.component.ts
```typescript
private buildForm() {
  this.accountsForm = this.fb.group({
    amount: ['', Validators.required],
    currency: ['', Validators.required],
    description: ['', [
      Validators.required, 
      Validators.minLength(8)
    ]],
    isToday: [true],
    date: [PrivateUtilsService.getCurrentDate(), Validators.required]
  });
}
```

A estas validaciones integradas se puede añadir otras creadas por el programador. Incluso con ejecución asíncrona para validaciones realizadas en el servidor.

Por ejemplo podemos agregar una validación específica a la descripción

```typescript
description: ['', [
  Validators.required,
  Validators.minLength(8),
  this.validateDescription
]]
```
Lo único que se necesita es una función que reciba como argumento el control a validar. El resultado debe ser un null si todo va bien. Y cualquier otra cosa en caso de fallo.

#### bhd-accounts-transaction.component.ts
```typescript
private validateDescription(control: AbstractControl) {
  const pattern = new RegExp('^[A-Z]+$', 'i');
  const description = control.value;
  let error = null;
  if (!pattern.test(description)) {
    error = { ...error, pattern: 'Must be just letters' };
  }
  return error;
}
```

## Estados de cambio y validación

Una vez establecidas las reglas, es hora de aplicarlas y avisar al usuario en caso de que se incumplan. Los formularios y controles reactivos están gestionados por máquinas de estados que determinan en todo momento la situación de cada control y del formulario en si mismo.

## Estados de validación

Al establecer una o más reglas para uno o más controles activamos el sistema de chequeo y control del estado de cada control y del formulario en su conjunto.

La máquina de estados de validación contempla los siguientes estados mutuamente excluyentes:

**VALID**: el control ha pasado todos los chequeos.

**INVALID**: el control ha fallado al menos en una regla.

**PENDING**: el control está en medio de un proceso de validación.

**DISABLED**: el control está desactivado y exento de validación.

Cuando un control incumple con alguna regla de validación, estas se reflejan en su propiedad errors que será un objeto con una propiedad por cada regla insatisfecha y un valor o mensaje de ayuda guardado en dicha propiedad.

## Estados de modificación

Los controles, y el formulario, se someten a otra máquina de estados que monitoriza el valor del control y sus cambios.

La máquina de estados de cambio contempla entre otros los siguientes:

**PRINSTINE**: el valor del control no ha sido cambiado por el usuario.

**DIRTY**: el usuario ha modificado el valor del control.

**TOUCHED**: el usuario ha tocado el control lanzando un evento blur al salir.

**UNTOUCHED**: el usuario no ha tocado el control.

Como en el caso de los estados de validación, el formulario también se somete a estos estados en función de cómo estén sus controles.

Veamos primero en el caso general del formulario. Uno de los usos más inmediatos es deshabilitar el botón de envío cuando la validación de algún control falla.

#### register.component.html
```html
<button (click)="sendForm()"
    [disabled]="accountsForm.invalid">Send money!</button>
```

#### bhd-accounts-transaction.component.ts
```typescript
public sendForm() {
  const transaction = this.accountsForm.value;
  console.log(transaction);
}
```
La validación particular para cada control permite informar al usuario del fallo concreto. Es una buena práctica de usabilidad el esperar a que edite un control antes de mostrarle el fallo. Y también es muy habitual usar la misma estrategia para cada control.

Lo que no queremos es llevar de vuelta la lógica a la vista; así que lo recomendado es crear una función auxiliar para mostrar los errores de validación.

#### bhd-accounts-transaction.component.ts
```typescript
public getError(controlName: string): string {
  let error = '';
  const control = this.formGroup.get(controlName);
  if (control.touched && control.errors != null) {
    error = JSON.stringify(control.errors);
  }
  return error;
}
```

En la vista colocaremos adecuadamente los mensajes para facilitarle la corrección al usuario.

#### bhd-accounts-transaction.component.ts
```html
<span>{{ getError('amount')}}</span>
<span>{{ getError('description')}}</span>
```

De igual forma es posible que al enviar el formulario se requieran hacer validacion extras.

#### bhd-accounts-transaction.component.ts
```typescript
public sendForm() {
  const { amount } = this.accountsForm.value;
  if (!(MathService.bhdParseFloat(amount) > 0)) {
    this.errorHandlerService.showToast('pleaseSelectAmount', ErrorTypes.ERROR);
    return;
  }
}
```
## Resumen
En resumen, los formularios reactivos tienen una API más robusta para agregar reactividad con RxJS, darle mejor mantenimiento a los formularios, separar la lógica en JavaScript del template o estructura en HTML, una forma más sencilla de agregar pruebas unitarias… En consecuencia, tus formularios con Angular tendrán una gran interfaz de usuario y UX.