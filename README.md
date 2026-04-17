```
import express from "express"
import cors from "cors"
import mysql from "mysql2/promise"

export const app = express()
app.use(express.json())
app.use(cors())

let con = await mysql.createConnection({
    host:"localhost",
    port:3306,
    database:"bekak",
    user:"root",
    password:""
})

async function getBekak(req, res){
    try{
    let sql = "select * from fajok order by nev"
    let [ adat ] = await con.execute(sql, [])
    res.send(adat)
     } catch (error) {
        console.log(error)
        res.status(500).send({msg:"Adatbázis hiba!"})
    }
}

async function getFigyelok(req, res){
    try{
    let sql = "select distinct * from megfigyelesek order by megfigyelo"
    let [ adat ] = await con.execute(sql, [])
    res.send(adat)
     } catch (error) {
        console.log(error)
        res.status(500).send({msg:"Adatbázis hiba!"})
    }
}

async function getKilatta(req, res){
    let {bekaid} = req.params
    try{
    let sql = "select * from megfigyelesek where bekaid = ? order by datum desc "
    let [ adat ] = await con.execute(sql, [bekaid])
    console.log(adat)
    if(adat.length == 0) res.status(404).send({error:"Ilyen békát nem látott senki!"})
    else res.send(adat)
     } catch (error) {
        console.log(error)
        res.status(500).send({msg:"Adatbázis hiba!"})
    }
}
async function postMegfigyeles(req, res){
    let {bekaid, helyszin, datum, megfigyelo, egyedszam} = req.body
    try{
    if (bekaid && helyszin&& datum&& megfigyelo&& egyedszam){
        let sql = "insert into megfigyelesek set bekaid = ?, helyszin = ?, datum = ?, megfigyelo = ?, egyedszam = ?"
        let [adat] = await con.execute(sql, [bekaid, helyszin, datum, megfigyelo, egyedszam])
        res.status(201).send(adat)
    }   else{
        res.status(400).send({error:"Hibás paraméterek!"})
    } } catch (error) {
        console.log(error)
        res.status(500).send({msg:"Adatbázis hiba!"})
    }
}

async function patchMegfigyeles(req, res){
    let {id, szam} = req.params
    try{
    if (id && szam){
        let sql = "update megfigyelesek set egyedszam = ? where id = ?"
        let [adat] = await con.execute(sql, [szam, id])
        if(adat.affectedRows == 0) return res.status(404).send({error:"Nem létező ID!"})
        res.status(201).send(adat)
    }   else{
        res.status(400).send({error:"Hibás paraméterek!"})
    } } catch (error) {
        console.log(error)
        res.status(500).send({msg:"Adatbázis hiba!"})
    }
}

async function deleteMegfigyeles(req, res){
    let {id} = req.params
    let sql = "delete from megfigyelesek where id=?"
    let [adat] = await con.execute(sql, [id])
    if(adat.affectedRows == 0) return res.status(404).send({error:"Nem létező ID!"})
    res.status(200).send(adat)
}

app.get("/", (req, res) => res.send("<h1>Békák v1.0.0</h1>"))
app.get("/bekak", getBekak)
app.get("/figyelok", getFigyelok)
app.get("/kilatta/:bekaid", getKilatta)
app.post("/megfigyeles", postMegfigyeles)
app.patch("/egyedszam/:id/:szam", patchMegfigyeles)
app.delete("/megfigyeles/:id", deleteMegfigyeles)

app.listen(88, err => console.log(err ? err : "Server on #88"))
```
