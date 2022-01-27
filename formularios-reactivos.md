# Formularios reactivos

Los Formularios Reactivos nos proveen de una manera de manejar las entradas de datos del usuario cuyos valores cambian en el tiempo.

Cada cambio que ocurre en el formulario devuelve un nuevo estado, lo que ayuda a mantener la integridad del modelo entre cada cambio. Los formularios reactivos están basados en flujos de datos de tipo Observable, donde cada entrada y cada valor toman la forma de un flujo de datos que puede ser accedido de manera asíncrona.

Los Formularios Reactivos son mas escalables, reusables y fáciles de probar. Cada Elemento de la vista está directamente enlazado al modelo mediante una instancia de FormControl. Las actualizaciones de la vista al modelo y del modelo a la vista son síncronas y no dependen de la representación en la Interfaz de Usuario del cliente.

## Form Builder

Entra en acción el FormBuilder, un servicio del que han de depender los componentes que quieran desacoplar el modelo de la vista. Se usa para construir un formulario creando un FormGroup, (un grupo de controles) que realiza un seguimiento del valor y estado de cambio y validez de los datos.

Para poder usarlo tenemos que importar el módulo de Angular en el que viene declarado, el ReactiveFormModule.

#### security.model.ts
```typescript

import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  declarations: [RegisterComponent],
  imports: [
    CommonModule,
    SecurityRoutingModule,
    ReactiveFormsModule
  ]
})
export class SecurityModule { }
```

Veamos un ejemplo de su declaración.

#### register.component.ts
```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

export class RegisterComponent implements OnInit {
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

#### register.component.ts
```typescript
private buildForm() {
  const ege = 25;
  const name = 'JOHN DOE';
  const dni = 1090412356;
  const email = 'john@angular.io';
  this.formGroup = this.formBuilder.group({
    ege: ege,
    name: name.toLowerCase(),
    dni: dni,
    country: '',
    email: email,
    password: ''
  });
}
```

Como ves, es fácil asignar valores por defecto. Incluso es un buen momento para modificar o transformar datos previos para ajustarlos a cómo los verá el usuario; sin necesidad de cambiar los datos de base.

## Form view

Mientras tanto en la vista html… Este trabajo previo y extra que tienes que hacer en el controlador se recompensa con una mayor limpieza en la vista. Lo único necesario será asignar por nombre el elemento html con el control typescript que lo gestionará.

Para ello usaremos dos directivas que vienen dentro del módulo reactivo son **[formGroup]="objetoFormulario"** para el formulario en su conjunto, y **formControlName="nombreDelControl"** para cada control.

#### register.component.html
```html
<form [formGroup]="formGroup">
  <label for="ege">Ege</label>
  <input name="ege"
        formControlName="ege"
        type="number" />
  <label for="name">Name</label>
  <input name="name"
        formControlName="name"
        type="text" />
  <label for="dni">Dni</label>
  <input name="dni"
        formControlName="dni"
        type="number" />
  <label for="country">Country</label>
  <input name="country"
        formControlName="country"
        type="text" />
  <label for="email">E-mail</label>
  <input name="email"
        formControlName="email"
        type="email" />
  <label for="password">Password</label>
  <input name="password"
        formControlName="password"
        type="password" />
</form>
```

## Validación y estados

La validación es una pieza clave de la entrada de datos en cualquier aplicación. Es el primer frente de defensa ante errores de usuarios; ya sean involuntarios o deliberados.

Dichas validaciones se solían realizar agregando atributos html tales como el conocido required. Pero todo eso ahora se traslada a la configuración de cada control, donde podrás establecer una o varias reglas de validación sin usar html.

## Validadores predefinidos y personalizados

Como veremos, tenemos la posibilidad de escribir nuestras propias validaciones para resolver diferentes casos y de igual forma se pueden usar las validaciones que vienen predefinidas como funciones en el objeto Validators de framework.

#### register.component.ts
```typescript
private buildForm() {
  const ege = 25;
  const name = 'JOHN DOE';
  const dni = 1090412356;
  const email = 'john@angular.io';
  const minPassLength = 4;

  this.formGroup = this.formBuilder.group({
    ege: ege,
    name: [name.toLowerCase(), Validators.required],
    dni: [dni, Validators.required],
    country: '',
    email: ['john@angular.io', [
      Validators.required, 
      Validators.email
    ]],
    password: ['', [
      Validators.required, 
      Validators.minLength(minPassLength)
    ]]
  });
}
```

A estas validaciones integradas se puede añadir otras creadas por el programador. Incluso con ejecución asíncrona para validaciones realizadas en el servidor.

Por ejemplo podemos agregar una validación específica a las contraseñas

```typescript
password: ['', [
  Validators.required,
  Validators.minLength(minPassLength),
  this.validatePassword
]]
```
Lo único que se necesita es una función que recibe como argumento el control a validar. El resultado debe ser un null si todo va bien. Y cualquier otra cosa en caso de fallo.

#### register.component.ts
```typescript
private validatePassword(control: AbstractControl) {
  const password = control.value;
  let error = null;
  if (!password.includes('$')) {
    error = { ...error, dollar: 'needs a dollar symbol' };
  }
  if (!parseFloat(password[0])) {
    error = { ...error, number: 'must start with a number' };
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

**UNTOUCHED**: el usuario no ha tocado y salido del control lanzando ningún evento blur.

Como en el caso de los estados de validación, el formulario también se somete a estos estados en función de cómo estén sus controles.

Veamos primero en el caso general del formulario. Uno de los usos más inmediatos es deshabilitar el botón de envío cuando la validación de algún control falla.

#### register.component.html
```html
<button (click)="register()"
    [disabled]="formGroup.invalid">Register me!</button>
```

#### register.component.ts
```typescript
public register() {
  const user = this.formGroup.value;
  console.log(user);
}
```
La validación particular para cada control permite informar al usuario del fallo concreto. Es una buena práctica de usabilidad el esperar a que edite un control antes de mostrarle el fallo. Y también es muy habitual usar la misma estrategia para cada control.

Lo que no queremos es llevar de vuelta la lógica a la vista; así que lo recomendado es crear una función auxiliar para mostrar los errores de validación.

#### register.component.ts
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

#### register.component.html
```html
<span>{{ getError('name')}}</span>
<span>{{ getError('email')}}</span>
<span>{{ getError('password')}}</span>
```

## Resumen
En resumen, los formularios reactivos tienen una API más robusta para agregar reactividad con RxJS, darle mejor mantenimiento a los formularios, separar la lógica en JavaScript del template o estructura en HTML, una forma más sencilla de agregar pruebas unitarias… En consecuencia, tus formularios con Angular tendrán una gran interfaz de usuario y UX.