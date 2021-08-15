# Useful Code Snippet 

make sure you also include reference url.

## HOF (loading spinner)

to abstract the loading spinner logic and apply this HOF for any component need this feature.

```
private fun launchDataLoad(block: suspend () -> Unit): Job {
   return viewModelScope.launch {
       try {
           _spinner.value = true
           block()
       } catch (error: TitleRefreshError) {
           _snackBar.value = error.message
       } finally {
           _spinner.value = false
       }
   }
}
```

usage:

```
fun refreshTitle() {
   launchDataLoad {
       repository.refreshTitle()
   }
}
```

ref: https://developer.android.com/codelabs/kotlin-coroutines#10

## 

