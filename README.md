# Tailwindcss <!-- omit in toc -->

Es un framework de CSS basado en clases de utilidad. A diferencia de otros frameworks como Bootstrap o Material UI, que ofrecen componentes prediseñados, Tailwindcss nos permite un control total al diseñar componentes e interfaces, permitiendo una mayor flexibilidad y personalización.

## Contenidos <!-- omit in toc -->

- [Instalación](#instalación)
- [Primeros pasos](#primeros-pasos)
  - [Experiencia de desarrollo](#experiencia-de-desarrollo)
  - [Agregando estilos customizados](#agregando-estilos-customizados)
    - [Modificando nuestro theme](#modificando-nuestro-theme)
    - [Arbitrary values](#arbitrary-values)
    - [Utilizando CSS y `@layer`](#utilizando-css-y-layer)
    - [Agregando plugins](#agregando-plugins)
- [Problemas frecuentes](#problemas-frecuentes)
  - [Interpolación de clases](#interpolación-de-clases)
  - [¿Cómo podemos extender las clases de los componentes?](#cómo-podemos-extender-las-clases-de-los-componentes)
    - [Tailwind Merge al rescate.](#tailwind-merge-al-rescate)
- [Creando un componente sin perder legibilidad](#creando-un-componente-sin-perder-legibilidad)
  - [Class Variance Authority](#class-variance-authority)

## Instalación

_Aclaración: en esta guía estaremos utilizando `pnpm` como gestor de paquetes. Para instalarlo puedes correr el siguiente comando:_

```bash
npm i -g pnpm
```

[Ir a la documentación ↗️](https://tailwindcss.com/docs/installation)

Para poder utilizar Tailwindcss en nuestros proyectos, tenemos que ejecutar los siguientes comandos:

```bash
pnpm add -D -E tailwindcss postcss autoprefixer
pnpx tailwindcss init -p --ts
```

Esto instalará las dependencias en nuestro proyecto e inicializará una configuración. Para que Tailwindcss funcione correctamente, debemos ir al archivo `tailwind.config` y agregar lo siguiente:

```ts
// ./tailwind.config.ts

import type { Config } from "tailwindcss";

export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx,mdx}"],
  theme: {
    extend: {},
  },
  plugins: [],
} satisfies Config;
```

_El array `content` especifica en que directorios hay archivos que contienen clases de Tailwindcss. Al hacer la build de nuestro proyecto, Tailwindcss tomará solo las clases de esos directorios, reduciendo el tamaño de nuestro bundle._

Por último, tenemos que agregar las siguientes directivas en nuestro archivo CSS global:

```css
/* src/index.css */

@tailwind base;
@tailwind components;
@tailwind utilities;
```

¡Listo! Ya tienes configurado Tailwindcss en tu proyecto. Para comprobar que funciona, vamos a agregar unas clases.

```tsx
// src/app.tsx

export default function App() {
  return (
    <div className="grid min-h-screen place-items-center">
      <h1 className="text-3xl text-indigo-500">Hello world!</h1>
    </div>
  );
}
```

## Primeros pasos

### Experiencia de desarrollo

Para facilitar aprender las clases de utilidad que nos brinda Tailwindcss (además de su [documentación](https://tailwindcss.com/docs)), tenemos la posiblidad de agregar IntelliSense a nuestro IDE de preferencia. Para _Visual Studio Code_, existe la extensión [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss).

_Si estamos trabajando en un equipo que utiliza `VSCode` podemos agregar la extensión dentro de la carpeta `.vscode` para que el workspace sugiera ese plugin a lxs nuevxs integrantes del equipo._

```diff
// .vscode/extensions.json

{
  "recommendations": [
    // ...
+   "bradlc.vscode-tailwindcss"
  ]
}
```

Otra herramienta que mejorará la experiencia de desarrollo es el plugin de [Prettier](https://github.com/tailwindlabs/prettier-plugin-tailwindcss). Este plugin ordena las clases de Tailwindcss para mayor consistencia.

Para utilizar el plugin, lo instalaremos como una dependencia de desarrollo:

```bash
pnpm add -D -E prettier-plugin-tailwindcss
```

Luego, agregamos el plugin en nuestra configuración de Prettier:

```diff
// .prettierrc.json

{
  "arrowParens": "always",
  "bracketSameLine": false,
  "bracketSpacing": true,
  "endOfLine": "auto",
  "jsxSingleQuote": true,
  "printWidth": 100,
  "quoteProps": "as-needed",
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "useTabs": false,
+ "plugins": ["prettier-plugin-tailwindcss"],
}
```

Si utilizamos funciones que reciban clases de tailwind y no tenemos el ordenamiento automático de Prettier, podemos agregar la siguiente configuración para recuperarlo:

```diff
// .prettierrc.json

{
  // ...
  "plugins": ["prettier-plugin-tailwindcss"],
+ "tailwindFunctions": ["cva"]
}
```

### Agregando estilos customizados

Tenemos distintas maneras de agregar o utilizar estilos que no vienen definidos en Tailwindcss. [Documentación ↗](https://tailwindcss.com/docs/adding-custom-styles)

#### Modificando nuestro theme

En nuestro archivo `tailwind.config` podremos sobreescribir o extender valores del theme que nos provee Tailwindcss. [Configuración por defecto ↗️](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js)

```ts
// ./tailwind.config.ts

import type { Config } from "tailwindcss";

import colors from "tailwindcss/colors";

export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx,mdx}"],
  // Manejo de dark mode por medio de clase. Por defecto utiliza "media"
  darkMode: "class",
  theme: {
    // Reduciendo la cantidad de colores del theme.
    colors: {
      red: "#ff0000",
      teal: colors.teal,
    },
    extend: {
      // Extendiendo los breakpoints.
      screens: {
        "3xl": "1600px",
      },
    },
  },
  plugins: [],
} satisfies Config;
```

#### Arbitrary values

Puede que en algunas situaciones necesitemos aplicar propiedades o valores que no se encuentran en las clases de utilidad de Tailwindcss. Para esos casos, podemos utilizar `arbitrary values`:

```tsx
// src/app.tsx

export default function App() {
  return <div className="grid grid-cols-[auto_1fr_auto]">{/* ... */}</div>;
}
```

Tailwindcss no tiene una utilidad para `grid-cols` que nos permita crear filas o columnas con tamaños distintos, pero podemos usar el prefijo `grid-cols-` junto con `[]` para agregar lo que queramos en la propiedad. El código se traduce en lo siguiente:

```css
grid-template-columns: auto 1fr auto;
```

<em>Los caracteres `_` son reemplazados por espacios.</em>

#### Utilizando CSS y `@layer`

Tenemos la posibilidad de crear estilos directamente desde los archivos `.css`

```css
/* src/index.css */

@tailwind base;
@tailwind components;
@tailwind utilities;

.my-custom-class {
  /* ... */
}
```

Si quisieramos aprovechar los beneficios de Tailwindcss, podemos extender cualquiera de los `@layer` que nos proporciona. De esta manera, cualquier clase que agreguemos, se podrá utilizar con los modificadores (ej: `hover:`, `lg:`, `before:`, etc) y _purgar esas clases en build time en el caso de que no las estemos utilizando_.

```css
/* src/index.css */

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  .my-custom-class {
    /* ... */
  }
}
```

Si queremos incluir alguna de las clases de Tailwindcss, podemos utilizar `@apply`. Vamos a crear una clase por temas de accesibilidad, la cual llamaremos `focus-ring`, que utilizaremos para identificar los elementos cuando se realice una navegación por teclado:

```css
/* src/index.css */

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  .focus-ring {
    @apply ring-2 ring-slate-900 ring-offset-2;
  }
}
```

De esta manera, cada vez que usemos la clase `focus-ring` aplicaremos todos esos estilos.

#### Agregando plugins

Al igual que con `@layer`, podemos agregar clases en el `tailwind.config` en la sección de plugins. El gran beneficio de agregarlo como plugin, es que **_tendremos autocompletado con TailwindCSS IntelliSense_**. [Configuración de plugins ↗️](https://tailwindcss.com/docs/plugins)

```ts
// ./tailwind.config.ts

import type { Config } from "tailwindcss";

import plugin from "tailwindcss/plugin";

export default {
  content: ["./index.html", "./src/**/*.{ts,tsx,mdx}"],
  theme: {
    extend: {},
  },
  plugins: [
    plugin(({ addUtilities }) => {
      addUtilities({
        ".focus-ring": {
          // De este modo podemos aplicar clases de Tailwindcss.
          "@apply ring-2 ring-slate-900 ring-offset-2": {},
        },
      });
    }),
  ],
} satisfies Config;
```

## Problemas frecuentes

A la hora de usar Tailwindcss, podemos enfrentarnos a estos inconvenientes:

### Interpolación de clases

Cuando nos encontramos con componentes que tienen variantes, es posible se nos ocurra realizar algo como esto:

```tsx
// src/components/button/index.tsx

import { forwardRef, type ComponentPropsWithoutRef } from "react";

interface Props extends ComponentPropsWithoutRef<"button"> {
  variant?: "primary" | "secondary";
}

export const Button = forwardRef<HTMLButtonElement, Props>(
  ({ className, type = "button", variant = "primary", ...props }, ref) => {
    return (
      <button
        className={`inline-flex h-10 touch-none select-none items-center justify-center gap-2 whitespace-nowrap rounded px-4 font-semibold text-slate-50 transition-opacity duration-300 hover:opacity-85 focus:outline-none focus-visible:focus-ring disabled:cursor-not-allowed disabled:opacity-50 bg-${
          variant === "primary" ? "indigo" : "red"
        }-500 ${className}`}
        ref={ref}
        type={type}
        {...props}
      />
    );
  }
);

Button.displayName = "Button";
```

Lo que tenemos en el ejemplo no funcionará, ya que Tailwindcss no detecta cuando se hace interpolación sobre una clase de utilidad. Para solucionar este problema, podríamos hacer lo siguiente:

```tsx
// src/components/button/index.tsx

import { forwardRef, type ComponentPropsWithoutRef } from "react";

interface Props extends ComponentPropsWithoutRef<"button"> {
  variant?: "primary" | "secondary";
}

export const Button = forwardRef<HTMLButtonElement, Props>(
  ({ className, type = "button", variant = "primary", ...props }, ref) => {
    const buttonVariants: Record<typeof variant, string> = {
      primary: "bg-indigo-500",
      secondary: "bg-slate-900",
    };

    return (
      <button
        className={`inline-flex h-10 touch-none select-none items-center justify-center gap-2 whitespace-nowrap rounded px-4 font-semibold text-slate-50 transition-opacity duration-300 hover:opacity-85 focus:outline-none focus-visible:focus-ring disabled:cursor-not-allowed disabled:opacity-50 ${buttonVariants[variant]} ${className}`}
        ref={ref}
        type={type}
        {...props}
      />
    );
  }
);

Button.displayName = "Button";
```

O si usamos una librería como `clsx` podríamos hacerlo de la siguiente forma:

```tsx
// src/components/button/index.tsx

import { forwardRef, type ComponentPropsWithoutRef } from "react";
import { clsx } from "clsx";

interface Props extends ComponentPropsWithoutRef<"button"> {
  variant?: "primary" | "secondary";
}

export const Button = forwardRef<HTMLButtonElement, Props>(
  ({ className, type = "button", variant = "primary", ...props }, ref) => {
    return (
      <button
        className={clsx(
          [
            // base
            "inline-flex h-10 touch-none select-none items-center justify-center gap-2 whitespace-nowrap rounded px-4",
            // font
            "font-semibold text-slate-50",
            // transitions
            "transition-opacity duration-300",
            // hover
            "hover:opacity-85",
            // disabled
            "disabled:cursor-not-allowed disabled:opacity-50",
            // accessibility
            "focus:outline-none focus-visible:focus-ring",
          ],
          {
            "bg-indigo-500": variant === "primary",
            "bg-slate-900": variant === "secondary",
          },
          className
        )}
        ref={ref}
        type={type}
        {...props}
      />
    );
  }
);

Button.displayName = "Button";
```

### ¿Cómo podemos extender las clases de los componentes?

Tenemos nuestro componente `Button` que utilizamos en nuestra app. ¿Qué pasa si quisieramos extender sus clases? Veamos el siguiente ejemplo:

```tsx
// src/app.tsx

import { Button } from "./components/button";

export default function App() {
  return (
    <div className="grid min-h-screen place-items-center">
      <Button className="p-10" variant="primary">
        Hello world!
      </Button>
    </div>
  );
}
```

En este ejemplo, lo único que estamos haciendo es consumir nuestro componente `Button` y modificar el padding para que ya no use más `px-4` que tiene por defecto y reemplazarlo por `p-10`. Pero, ¿por qué no funciona? En primer lugar, no estamos quitando de nuestro componente el `px-4`, solamente estamos agregando el `p-10` al final. Nuestro `button` en el DOM se ve así:

```html
<button
  class="inline-flex h-10 touch-none select-none items-center justify-center gap-2 whitespace-nowrap rounded px-4 font-semibold text-slate-50 transition-opacity duration-300 hover:opacity-85 focus:outline-none focus-visible:focus-ring disabled:cursor-not-allowed disabled:opacity-50 bg-indigo-500 p-10"
  type="button"
>
  Hello world!
</button>
```

Y en segundo lugar, por como funciona la cascada en CSS. Que agreguemos al final de nuestras clases `p-10` no garantiza que se va a sobreescribir, porque va a depender de como fueron escritas las clases en la hoja de estilos de Tailwindcss. Podría suceder algo como esto:

```css
.p-10 {
  padding: 2.5rem; /* 40px */
}

/* Resto de las utilidades de clase .p- */

.px-4 {
  padding-inline: 1rem; /* 16px */
}
```

#### Tailwind Merge al rescate.

[Repositorio en GitHub ↗️](https://github.com/dcastil/tailwind-merge?tab=readme-ov-file)

Es un pequeño paquete que viene a solucionar este problema. Para usarlo solo tenemos que instalarlo de la siguiente forma:

```bash
pnpm add -E tailwind-merge
```

Una vez instalado, podemos crear una función que combine `clsx` junto con `tailwind-merge` para seguir teniendo la posibilidad de utilizar clases de forma condicional.

```ts
// src/utils/cn.ts

import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export const cn = (...classNames: ClassValue[]) => {
  return twMerge(clsx(...classNames));
};
```

Solo nos quedará reemplazar la función `clsx` por `cn` en nuestra aplicación.

## Creando un componente sin perder legibilidad

Cuando creamos componentes que tengan muchas variantes (ej:botón), nuestro código puede volverse un tanto difícil de leer y mantener usando Tailwindcss.

### Class Variance Authority

Es una librería que nos facilita la forma de escribir componentes que contengan muchas variantes. [Ir a la documentación ↗️](https://cva.style/docs)

Para instalar `cva` en tu proyecto:

```bash
pnpm add -E class-variance-authority
```

Siguiendo con lo que venimos trabajando previamente _(clsx, tailwind-merge)_, vamos a hacer unas modificaciones una vez instalada `cva`.

```diff
// src/utils/cn.ts

- import { clsx, type ClassValue } from "clsx";
+ import { cx, type CxOptions } from "class-variance-authority";
import { twMerge } from "tailwind-merge";

- export const cn = (...classNames: ClassValue[]) => {
+ export const cn = (...classNames: CxOptions) => {
  return twMerge(cx(...classNames));
};
```

No hace falta instalar en nuestro proyecto `clsx` ya que `cva` lo utiliza internamente y nos expone `cx` como un abreviado de `clsx` y el type `CxOptions` como `ClassValue[]`.

Antes de comenzar a utilizar esta librería, vamos a configurar `VSCode` para que podamos mantener `Tailwind IntelliSense` dentro de la función `cva`. En nuestras configuraciones de `vscode` o si trabajamos en un proyecto grupal, en la carpeta `.vscode` agregaremos:

```diff
// .vscode/settings.json

{
  // ...
+ "tailwindCSS.experimental.classRegex": [
+   ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
+   ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
+ ]
}
```

Ahora si, veamos como quedaría un componente `button` utilizando `cva`:

```tsx
// src/components/button/index.tsx

import { forwardRef, type ComponentPropsWithoutRef } from "react";
import { cva, type VariantProps } from "class-variance-authority";

import { cn } from "../../utils/cn";

export const buttonStyles = cva(
  [
    // base
    "inline-flex h-10 touch-none select-none items-center justify-center gap-2 whitespace-nowrap rounded",
    // font
    "font-semibold",
    // transitions
    "transition-opacity duration-300",
    // disabled
    "disabled:cursor-not-allowed disabled:opacity-50",
    // accessibility
    "focus:outline-none focus-visible:focus-ring",
  ],
  {
    variants: {
      fullWidth: {
        true: "w-full",
      },
      rounded: {
        true: "rounded-full",
      },
      size: {
        sm: "h-9 px-2",
        base: "h-10 px-4",
        lg: "h-11 px-8",
      },
      variant: {
        primary: "bg-indigo-500 text-slate-100 hover:opacity-85",
        secondary: "bg-slate-900 text-slate-100 hover:opacity-85",
        outline: "border bg-transparent text-slate-900 hover:bg-slate-50",
      },
    },
    defaultVariants: {
      size: "base",
      variant: "primary",
    },
  }
);

type Props = ComponentPropsWithoutRef<"button"> &
  VariantProps<typeof buttonStyles>;

export const Button = forwardRef<HTMLButtonElement, Props>(
  (
    {
      className,
      fullWidth,
      rounded,
      size,
      type = "button",
      variant = "primary",
      ...props
    },
    ref
  ) => {
    return (
      <button
        className={cn(
          buttonStyles({ fullWidth, rounded, size, variant }),
          className
        )}
        ref={ref}
        type={type}
        {...props}
      />
    );
  }
);

Button.displayName = "Button";
```

Podemos ver que quitamos las clases de Tailwindcss fuera del componente y creamos la función `buttonStyles` utilizando `cva`. La función `cva` acepta dos parámetros:

- Los estilos por defecto que serán aplicados al componente (recordemos que utiliza `clsx`, por lo que podremos pasarle los estilos por defecto de diferentes formas).
- Un objeto donde definamos nuestras variantes. Se compone de la siguiente manera:
  - `variants`: definimos dentro de este objeto todas las variantes que tendrá nuestro componente. Lo que definamos en este objeto, serán los parámetros de la función `buttonStyles` y las `props` de nuestro componente.
  - `defaultVariants`: los valores por defecto de los parámetros de nuestra función `buttonStyles`.
  - `compoundVariants`: en el caso de que necesitemos más control, podemos definir estilos cuando se cumplan ciertas condiciones [Compound Variants ↗️](https://cva.style/docs/getting-started/variants#compound-variants).

Utilizaremos la función `buttonStyles` para extender nuestro type `Props`, agregando el tipado de todas las variantes previamente definidas, y darle estilos a nuestro `<Button />`. La ventaja de esto es que podremos usar `buttonStyles` por fuera del componente. Veamos el siguiente ejemplo:

Es muy común que en nuestras aplicaciones, los componentes `button` y `link` tengan un estilo similar, aunque su comportamiento sea distinto. Para no tener que entrar en el mundo de _polimorfismo de componentes_ podemos hacer lo siguiente.

```tsx
// src/app.tsx

import { buttonStyles } from "./components/button";

export default function App() {
  return (
    <a
      className={buttonStyles({ rounded: true, variant: "secondary" })}
      href="#"
    >
      Hello World!
    </a>
  );
}
```

De esta manera, nuestro `<a>` tendrá acceso a todas las variantes que definimos en nuestro `<Button />`.
