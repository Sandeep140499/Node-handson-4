const exp = require("express").Router()
const bcrypt = require('bcrypt')
const users = [] 
const jwtToken = require('jsonwebtoken')
const { check, validationResult } = require('express-validator') 

exp.post('/signup', [
    check("email", "please provide a valid mail")
        .isEmail(),
    check("password", "please provide a valid password greater than 6 character")
        .isLength({
            min: 6
        })
], async (req, res) => {
    const { email, password } = req.body
    // email and password from user

    const error = validationResult(req)
    if (!error.isEmpty()) {
        return res.status(400).json({
            error: error.array()
        });
    }
    
    let user = users.find((user) => {
        return user.email == email;
    })
    if (user) {
        res.status(400).json({
            "err": [
                {
                    "msg": "This user is already existed"
                }
            ]
        })
    }
    else if (!user) {
        res.status(400).json({
            "err": [
                {
                    "msg": "Registration successful"
                }
            ]
        })
    }
    const hashPassword = await bcrypt.hash(password, 10)
    users.push({
        email,
        password: hashPassword
    })
   
})

exp.post('/login', async (req, res) => {
    const { password, email } = req.body
    let user = users.find((user) => {   
        return user.email === email;
    })

    if (!user) {
        return res.status(400).json({
            "err": [
                {
                    "msg": "Invalid Credentials, Register first"
                }
            ]
        })
    }

    let match = await bcrypt.compare(password, user.password)
    if (!match) {
        return res.status(400).json({
            "err": [
                {
                    "massege": "Invalid Credentials"
                }
            ]
        })
    }

    const token = await jwtToken.sign({
        email
    }, "lkjasdf", {
        expiresIn: 36000
    })
    res.json({
        token
    })
})

exp.get('/api', (req, res) => {
    res.json(users)

})

module.exports = exp;
