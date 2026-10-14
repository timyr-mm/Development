# Job, Deferred, Билдеры

### Черновик
При каждом создании корутины, например через билдер `launch {...}`, происходит следующее: Создается новый `Job`, который помещается в контекст `CoroutineContext`, далее создается `Continuation` в котором хранится контекст. То есть `Continuation` — это обертка вокруг исполнения корутины. Она хранит всё, что нужно, чтобы продолжить выполнение после приостановки (suspend). Внутри неё есть контекст, а внутри контекста лежит `Job`.

Такой сценарий используется для каждой корутины. Если корутина является дочерней, то она свой контекст наследует от контекста родительской корутины и в ее `Job` будет ссылка на `Job` родительской корутины.


### Что такое Job?
Job — Четкого определения нет. Но можно сказать, что это дескриптор (управляющий объект) корутины в Kotlin, который позволяет управлять её жизненным циклом (через сам интерфейс Job): ожидать завершения, отменять, проверять состояние. Также он отвечает за взаимодействие с дочерними и родительскими корутинами.

### Какие бывают состояния у Job?

| **Состояние**                  | **isActive** | **isCompleted** | **isCancelled** |
|--------------------------------|--------------|-----------------|-----------------|
| New (optional initial state)   | false        | false           | false           |
| Active (default initial state) | true         | false           | false           |
| Completing (transient state)   | true         | false           | false           |
| Cancelling (transient state)   | false        | false           | true            |
| Cancelled (final state)        | false        | true            | true            |
| Completed (final state)        | false        | true            | false           |

**Интерфейс Job**
``` kotlin
public interface Job : CoroutineContext.Element {
    public companion object Key : CoroutineContext.Key<Job>

    @ExperimentalCoroutinesApi
    public val parent: Job?

    public val isActive: Boolean

    public val isCompleted: Boolean

    public val isCancelled: Boolean

    public fun start(): Boolean
	
    public fun cancel(cause: CancellationException? = null)

    public val children: Sequence<Job>

    public suspend fun join()
}
```

### Что такое Deferred?
Deferred — Наследник Job, который расширяет его для возможности получения результата работы корутины через функцию `await()`.

**Интерфейс Deferred<T> : Job**
``` kotlin
public interface Deferred<out T> : Job {
    public suspend fun await(): T

    public val onAwait: SelectClause1<T>

    @ExperimentalCoroutinesApi
    public fun getCompleted(): T

    @ExperimentalCoroutinesApi
    public fun getCompletionExceptionOrNull(): Throwable?
}
```

### Что такое билдер?
Билдер корутины — это функция, создающая и запускающая корутину на основе переданного блока кода.

### Какие бывают билдеры?
* launch

* async

* runBlocking

* withContext

* coroutineScope

* supervisorScope
