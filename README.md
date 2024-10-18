# FDV_Sprites

## **_1. Agregar el atlas de sprites del personaje a la escena. Configurar el sprite como múltiple y subdividirlo para tener acceso a los sprites para generar las animaciones. Agregar una de las imágenes a la escena._**

Creamos el proyecto de Unity eligiendo la plantilla `2D (Built-in Render Pipeline)`.

![1 Creación del proyecto](https://github.com/user-attachments/assets/b2470964-a82d-4ef9-9bca-37c501d19ad8)

Seleccionamos el atlas de sprites y desde el inspector cambiamos el `Sprite Mode` de `Single` a `Multiple`. Ahora hay que separar los distintos sprites que se encuentran dentro del atlas. Para ello, pulsamos el botón _"Sprite Editor"_ y luego en _"Slice"_, en la esquina superior izquierda. Ahora, en la pequeña ventana que aparece, cambiamos `Type` a `Grid By Cell Count` y ponemos el número de columnas y filas de nuestro atlas.

![image](https://github.com/user-attachments/assets/0d1ed033-41a7-486e-b4fb-a8df62e2920d)

![image](https://github.com/user-attachments/assets/b7cbb2b1-3402-4abe-a1a7-498fb63d611a)

Ahora que el atlas ha sido trozeado y se han generado los distintos sprites, añadiremos uno a la escena.

![image](https://github.com/user-attachments/assets/b98cdfd0-e1ac-451c-979d-c461dafb23a7)

## **_2. Agregar al personaje la animación “Walk Down”._**

Ahora crearemos una animación para el objeto. Desde el sprite atlas, seleccionamos el conjunto de sprites que compondrán nuestra animación y lo soltamos en la escena. Esto creará un GameObject con el componente `Animator` y una animación, la cual ya estará incluida en el `Animator`.

![image](https://github.com/user-attachments/assets/535abda4-0b4a-46c9-8999-40e202c8ca68)

![Ejercicio 2](https://github.com/user-attachments/assets/a066f40a-fd58-4371-9c4b-c7c901d89c4f)

## **_3. Crear los scripts para controlar el movimiento del personaje. En este caso sólo tendremos que mover las coordenadas (X, Y)._**

Creamos un script y lo añadimos al GameObject del personaje:

```c#
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    private float _horizontalMovement;
    private float _verticalMovement;
    private Vector3 _direction;
    
    [SerializeField] private float _moveSpeed = 10.0f;

    private void Update()
    {
        _horizontalMovement = Input.GetAxisRaw("Horizontal");
        _verticalMovement = Input.GetAxisRaw("Vertical");

        if (_horizontalMovement != 0 || _verticalMovement != 0)
        {
            _direction = new Vector3(_horizontalMovement, _verticalMovement, transform.position.z).normalized;
            transform.Translate(_direction * (_moveSpeed * Time.deltaTime));
        }
    }
}
```

![image](https://github.com/user-attachments/assets/0615b4da-82cf-40a8-afba-03596c2aa29d)


![Ejercicio3](https://github.com/user-attachments/assets/35ee98c7-f2ee-48cd-89e7-37a0ea11c08a)

## **_4. Orientar el sprite hacia donde camina._**

```c#
using System;
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    [SerializeField] private float _moveSpeed = 10.0f;
    
    private float _horizontalMovement;
    private float _verticalMovement;
    private Vector3 _direction;

    private SpriteRenderer _spriteRenderer;

    private void Start()
    {
        _spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void Update()
    {
        _horizontalMovement = Input.GetAxisRaw("Horizontal");
        _verticalMovement = Input.GetAxisRaw("Vertical");

        if (_horizontalMovement != 0 || _verticalMovement != 0)
        {
            _spriteRenderer.flipX = _horizontalMovement > 0;
            
            _direction = new Vector3(_horizontalMovement, _verticalMovement, transform.position.z).normalized;
            transform.Translate(_direction * (_moveSpeed * Time.deltaTime));
        }
    }
}
```

![Ejercicio 4](https://github.com/user-attachments/assets/6d24ace0-ddb5-4dc1-bb00-4f92734091b2)

## **_5. Crear las animaciones del personaje y crear un script en el que el personaje cambie la animación._**

Creamos una animación para camminar hacia arriba y otra para abajo. Además, crearemos dos variables desde la ventana _Animator_:
* `isWalking`, de tipo `bool` y que nos indicará si el personaje está moviéndose.
* `verticalMovement`, de tipo `float`. En caso de que su valor sea mayor que 0, activaremos la animación de mirar arriba. En el caso de que sea menor que 0, se activará la de mirar abajo.

![image](https://github.com/user-attachments/assets/af779083-aae5-40bd-902f-ec21d8f5eb0e)

Las condiciones para las transiciones son:
* `[Idle] --> [Walk_Lateral] when isWalking is true`
* `[Walk_Lateral] --> [Idle] when isWalking is false`

* `[Idle] --> [Walk_Up] when verticalMovement > 0`
* `[Walk_Up] --> [Idle] when isWalking is false`

* `[Idle] --> [Walk_Down] when verticalMovement < 0`
* `[Walk_Down] --> [Idle] when isWalking is false`

* `[Walk_Lateral] --> [Walk_Up] when verticalMovement > 0`
* `[Walk_Up] --> [Walk_Lateral] when isWalking is true AND verticalMovement < 1`

* `[Walk_Lateral] --> [Walk_Down] when verticalMovement < 0`
* `[Walk_Down] --> [Walk_Lateral] when isWalking is true AND verticalMovement > -1`

Ahora tan solo hay que modificar los valores de esas variables desde el script.

```c#
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    [SerializeField] private float _moveSpeed = 10.0f;
    
    private float _horizontalMovement;
    private float _verticalMovement;
    private Vector3 _direction;

    private SpriteRenderer _spriteRenderer;
    private Animator _animator;
    
    private static readonly int IsWalking = Animator.StringToHash("isWalking");
    private static readonly int VerticalMovement = Animator.StringToHash("verticalMovement");

    private void Start()
    {
        _spriteRenderer = GetComponent<SpriteRenderer>();
        _animator = GetComponent<Animator>();
    }

    private void Update()
    {
        _horizontalMovement = Input.GetAxisRaw("Horizontal");
        _verticalMovement = Input.GetAxisRaw("Vertical");

        _animator.SetFloat(VerticalMovement, _verticalMovement);
        if (_horizontalMovement != 0 || _verticalMovement != 0)
        {
            _animator.SetBool(IsWalking, true);
            _spriteRenderer.flipX = _horizontalMovement > 0;
            
            _direction = new Vector3(_horizontalMovement, _verticalMovement, transform.position.z).normalized;
            transform.Translate(_direction * (_moveSpeed * Time.deltaTime));
        }
        else
        {
            _animator.SetBool(IsWalking, false);
        }
    }
}
```

![Ejercicio 5](https://github.com/user-attachments/assets/c6a2710e-946f-4b82-ab30-b18aa5524975)
