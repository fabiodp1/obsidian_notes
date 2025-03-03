[Your React App Isn’t Great — Here’s Why. | by Pierre G. | Dec, 2024 | Level Up Coding](https://levelup.gitconnected.com/your-react-app-isnt-great-here-s-why-5eb61b3f110b)

# Code splitting
[Code-Splitting – React](https://legacy.reactjs.org/docs/code-splitting.html)

Per ottimizzare le performance è importante che fra componenti le callback vengano passate utilizzando l'hook [[useCallback]], in modo da evitare re-render inutili.
Inoltre se il nostro componente è un context provider, è buona pratica passare come value la versione memoized:

```ts
function AuthProvider(props) {
  const {
    data: user,
    error,
    ...
  } = useAsync()

  const login = React.useCallback(
    form => auth.login(form).then(user => setData(user)),
    [setData],
  )

  const register = React.useCallback(
    form => auth.register(form).then(user => setData(user)),
    [setData],
  )

  const logout = React.useCallback(() => {
    auth.logout()
    setData(null)
  }, [setData])

  const value = React.useMemo(() => ({user, login, logout, register}), [
    login,
    logout,
    register,
    user,
  ])

  if (isLoading || isIdle) {
    return <FullPageSpinner />
  }

  if (isError) {
    return <FullPageErrorFallback error={error} />
  }

  if (isSuccess) {
    return <AuthContext.Provider value={value} {...props} />
  }

  throw new Error(`Unhandled status: ${status}`)
}
```