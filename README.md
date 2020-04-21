# Responsive-Register-and-Login-Page

Always in Register and Login Pages we first should have auth reducer

## Register

Register.js

```
import React, { Component } from 'react'
import { Link } from 'react-router-dom'

class Register extends Component {
    state = {
        username: '',
        email: '',
        password: '',
        password2: ''
    }

    onSubmit = e => {
        e.preventDefault()
    }

    onChange = e => {
        const { value, name } = e.target
        this.setState({ [name]: value })
    }

    render() {
        const { username, email, password, password2 } = this.state
        return (
            <div className='col-md-6 m-auto'>
                <div className='card card-body mt-5'>
                    <h2 className='text-center'>Register</h2>
                    <form onSubmit={this.onSubmit}>
                        <div className='form-group'>
                            <label>Username</label>
                            <input
                                type='text'
                                className='form-control'
                                name='username'
                                onChange={this.onChange}
                                value={username}
                            />
                        </div>
                        <div className='form-group'>
                            <label>Email</label>
                            <input
                                type='email'
                                className='form-control'
                                name='email'
                                onChange={this.onChange}
                                value={email}
                            />
                        </div>
                        <div className='form-group'>
                            <label>Password</label>
                            <input
                                type='password'
                                className='form-control'
                                name='password'
                                onChange={this.onChange}
                                value={password}
                            />
                        </div>
                        <div className='form-group'>
                            <label>Confirm Password</label>
                            <input
                                type='password'
                                className='form-control'
                                name='password2'
                                onChange={this.onChange}
                                value={password2}
                            />
                        </div>
                        <div className='form-group'>
                            <button type='submit' className='btn btn-primary'>
                                Register
                            </button>
                        </div>
                        <p>
                            Already have an account? <Link to='/login'>Login</Link>
                        </p>
                    </form>
                </div>
            </div>
        )
    }
}

export default Register
```

Login.js

```
import React, { Component } from 'react'
import { Link } from 'react-router-dom'

class Login extends Component {
    state = {
        username: '',
        password: ''
    }

    onSubmit = e => {
        e.preventDefault()
    }

    onChange = e => {
        const { value, name } = e.target
        this.setState({ [name]: value })
    }

    render() {
        const { username, password } = this.state
        return (
            <div className='col-md-6 m-auto'>
                <div className='card card-body mt-5'>
                    <h2 className='text-center'>Login</h2>
                    <form onSubmit={this.onSubmit}>
                        <div className='form-group'>
                            <label>Username</label>
                            <input
                                type='text'
                                className='form-control'
                                name='username'
                                onChange={this.onChange}
                                value={username}
                            />
                        </div>
                        <div className='form-group'>
                            <label>Password</label>
                            <input
                                type='password'
                                className='form-control'
                                name='password'
                                onChange={this.onChange}
                                value={password}
                            />
                        </div>
                        <div className='form-group'>
                            <button type='submit' className='btn btn-primary'>
                                Login
                            </button>
                        </div>
                        <p>
                            Don't have an account? <Link to='/register'>Register</Link>
                        </p>
                    </form>
                </div>
            </div>
        )
    }
}

export default Login
```

auth.js (reducer)

```
const initialState = {
    token: localStorage.getItem('token'),
    isAuthenticated: null,
    isLoading: false,
    user: null
}

export default function (state = initialState, action) {
    switch (action.type) {
        default:
            return state
    }
}
```

components/common/PrivateRoute.js

```
import React from 'react'
import { Route, Redirect } from 'react-router-dom'
import { connect } from 'react-redux'

function PrivateRoute({ component: Component, auth, ...rest }) {
    return (
        <Route
            {...rest}
            render={props => {
                if (auth.isLoading) {
                    return <h2>Loading...</h2>
                } else if (!auth.isAuthenticated) {
                    return <Redirect to='/login' />
                } else {
                    return <Component {...props} />
                }
            }}
        />
    )
}

const mapStateToProps = state => {
    return {
        auth: state.auth
    }
}

export default connect(mapStateToProps)(PrivateRoute)
```

Now inside App.js instead of using Route, use PrivateRoute component


Next thing I want to do is, I want start to implement some actions, specifically the loadUser action, because we need to constantly check to see if the user is Authenticated or not. Because we're dealing with REST API.

So, what we'll do, is we'll have the componentDidMount in our app component and we call the loadUser action, which is gonna make a request to the api/auth/user with a token. and then basically return isAuthenticated = true along with any other data.

actionType.js:

```
export const USER_LOADING = 'USER_LOADING'
export const USER_LOADED = 'USER_LOADED'
export const AUTH_ERROR = 'AUTH_ERROR'
```

auth.js (reducer):

```
import { USER_LOADED, USER_LOADING, AUTH_ERROR } from '../actions/types'

const initialState = {
    token: localStorage.getItem('token'),
    isAuthenticated: null,
    isLoading: false,
    user: null
}

export default function (state = initialState, action) {
    switch (action.type) {
        case USER_LOADING:
            return {
                ...state,
                isLoading: true
            }
        case USER_LOADED:
            return {
                ...state,
                isAuthenticated: true,
                isLoading: false,
                user: action.payload
            }
        case AUTH_ERROR:
            localStorage.removeItem('token')
            return {
                ...state,
                token: null,
                user: null,
                isAuthenticated: false,
                isLoading: false
            }
        default:
            return state
    }
}
```

auth.js (action):

```
import axios from 'axios'
import {
    USER_LOADED,
    USER_LOADING,
    AUTH_ERROR
} from './types.js'

// CHECK TOKEN & LOAD USER
export const loadUser = () => (dispatch, getState) => {
    // User Loading 
    dispatch({
        type: USER_LOADING
    })

    // Get Token from state
    const token = getState().auth.token;

    // Headers
    const config = {
        headers: {
            'Content-Type': 'application/json'
        }
    }

    // If token, add to headers config
    if (token) {
        config.headers['Authorization'] = `Token ${token}`
    }

    // Making Request to get the user
    axios.get('/api/auth/user', config)
        .then(res => {
            dispatch({
                type: USER_LOADED,
                payload: res.data
            })
        })
        .catch(err => {
            console.log(err)
        })
}
```

Then import auth action (loadUser) to App.js file and use it inside componentDidMount:

App.js:

```
```
