# Sql x NoSql

### Criar uma tabela ###
###### *`create table | primary | tag3 |`*
---
*SQL*
~~~sql
create table 
  people(
    id not null auto_incremento,
    user_id varchar(30) 
    age Number,
    status Char(1),
  primary key id
)
~~~
*MongoDB*
~~~javascript
???
~~~
---
> ### Inserir um único registro ### 
> [!TIP]
> Helpful
###### `insert | into | values | insertOne |`

nota[^1],

*SQL*
~~~sql
insert into people(user_id, age, status) values('abc123',55,'A')
~~~
*MongoDB*
~~~javascript
db.people.insertOne({uer_id:'abc123', age:25, status: 'A'})
~~~
---
### Inserir múltiplos registros ### 
###### `insert into | values | insertMany |`
---
*SQL*
~~~sql
insert into people(user_id, age, status) values('bcd234',13,'B');
insert into people(user_id, age, status) values('cde345',31,'C');
insert into people(user_id, age, status) values('def456',23,'D');
~~~
*MongoDB*
~~~javascript
db.people.insertMany([
    {user_id:'bcd234', age:13, status: 'B'},
    {user_id:'cde345', age:31, status: 'C'},
    {user_id:'def456', age:23, status: 'D'},
])
~~~
---


//###################################################################
//################# CAPITULO 1: Tabelas e Indices ###################
//###################################################################




// Listando todos os documentos da coleção
db.people.find()

// Adicionar novo campo na coleção]
// alter table people
// add join_date datetime

// update people set join_date = getdate()
db.people.updateMany(
    {}, //Aqui entraria o seu where
    {
        $set:{
            join_date: new Date()
        }
    }
)

// Criar Índice nas collections
// Índice simples:
// create index idx_user_id_asc
// on people(user_id)

// Índice composto:
// create index idx_user_id_asc_age_desc
// on people(user_id, age DESC)
db.people.createIndex({user_id:1})
db.people.createIndex({user_id:1, age:-1})

// Deletar documentos da collection
// delete from people
db.people.deleteOne({})
db.people.deleteMany({})

// Apagar a tabela.
// drop table people
db.people.drop()

//###################################################################
//###################### CAPITULO 2: Queries ########################
//###################################################################

// select * from people
db.people.find()

// select user_id, status from people
db.people.find({}, {user_id:1, status:1})

// select user_id, status from people --sem exibir o _id
db.people.find({}, {user_id:1, status:1, _id:0})

// select * from people where status = 'A'
db.people.find({status: 'A'})

// select user_id, status from people where status = 'A'
db.people.find({status: 'A'}, {user_id:1, status:1, _id:0})

// select  user_id, status from people where status <> 'A'
db.people.find({status: {$ne:'A'}}, {user_id:1, status:1, _id:0})

// select  user_id, status from people where status <> 'A' and age = 50
db.people.find({status: {$ne:'A'}, age:50}, {user_id:1, status:1, _id:0})

// select  user_id, age, status from people where status = 'A' and age > 25 
db.people.find({status: {$ne:'A'}, age:{$gt:25}}, {user_id:1, age:1, status:1, _id:0})

// select  user_id, age, status from people where status = 'A' and age <= 55 
db.people.find({status: {$ne:'A'}, age:{$lte:55}}, {user_id:1, age:1, status:1, _id:0})

// select * from people where age < 25 and age <= 55
db.people.find({status: {$ne:'A'}, age:{$gt:25, $lte:55}})

// select user_id, age, status from people where user_id like '%bc%'
db.people.find({user_id:/bc/}, {user_id:1, age:1, status:1, _id:0})

// select user_id, age, status from people where user_id like 'bc%'
db.people.find({user_id:/^bc/}, {user_id:1, age:1, status:1, _id:0})

//select * from people where status = 'A' order by user_id ASC
db.people.find({status: 'A'}).sort({user_id:1})

//select * from people where status <> 'A' order by user_id ASC
db.people.find({status: {$ne:'A'}}).sort({user_id:1})

//select * from people where status in ('A', 'B')
db.people.find({status: {$in:['A', 'B']}}).sort({user_id:1})

//select * from people where status not in ('A', 'B')
db.people.find({status: {$nin:['A', 'B']}}).sort({user_id:1})

//select * from people where age = 13 or status = 'A'
db.people.find({$or: [{status:'A'}, {age:13}]})

// select count(*) from people
db.people.countDocuments()

// select count(user_id) from people
db.people.countDocuments({user_id: {$exists: true}})

// select count(*) from people where age > 20
db.people.countDocuments({age: {$gt: 20}})

// select * from distinct(status) from people
db.people.distinct('status')

// select * from people offset 10 rows fetch next 5
db.people.find().skip(10).limit(5)

//###################################################################
//###################### CAPITULO 3: Update #########################
//###################################################################

// update people set status = 'C' where age = 25
db.people.updateMany({age: 25},{$set:{status:'C'}})

// update people set age = age + 3 where status = 'C'
db.people.updateMany({status:'C'},{$inc:{age:3}})
   


db.people.find()

//###################################################################
//###################### CAPITULO 3: Delete #########################
//###################################################################

// delete from people where stuts = 'D'
db.people.deleteOne({status:'D'})
db.people.deleteMany({status:'D'})


//###################################################################
//###################### CAPITULO 3: Aggregate ######################
//###################################################################

// select 
//     top 10 
//     age as idade, 
//     status, 
//     user_id as id 
// from 
//     people 
// where
//     status = 'A'
// order By
//     user_id ASC

db.people.aggregate([
    // estágio 1
    { $match: {status: 'C'}},
    // estágio 2
    {
        $project: {
          idade: '$age',
          status: 1,
          id: '$user_id',
          _id:0
        }
    },
    // estágio 3
    { $sort: {
      id:1
    }},
    // estágio 4
    {$limit: 10},
])

// CREATE VIEW
db.createView('people_view', 'people', [
    // estágio 1
    { $match: {status: 'C'}},
    // estágio 2
    {
        $project: {
          idade: '$age',
          status: 1,
          id: '$user_id',
          _id:0
        }
    },
    // estágio 3
    { $sort: {
      id:1
    }},
    // estágio 4
    {$limit: 10},
])

db.people_view.find()

db.people_view.aggregate([
    {$match: {
      status: 'C'
    }}
])

db.people_view.drop()


/*
insert into people_collection_2
select age AS idade, status, user_id as id
from people
*/

db.people.aggregate([
    { $match: {status: 'C'}},
    {
        $project: {
          idade: '$age',
          status: 1,
          id: '$user_id',
          _id:0
        }
    },
    { $sort: {id:1}},
    {$limit: 10},
    {$out: 'people_collection_2'}
])

db.people_collection_2.find()

db.people_view.aggregate([
    {
        $unionWith: {
          coll: 'people_collection_2'
        }
    }
])

db.people_view.find()

db.products.insertMany([
    {name:'Soap', price: 5, category: 'Hygiene'},
    {name:'ToothPaste', price: 3.5, category: 'Hygiene'},
    {name:'Coca Cola', price: 8.75, category: 'Drinks'},
    {name:'7-Up', price: 6, category: 'Drinks'},
    {name:'CornFlakes', price: 19.9, category: 'Cereals'},
    {name:'Banana', price: 8.9, category: 'Fruits'},
    {name:'Apple', price: 12.3, category: 'Fruits'},
    {name:'Orange', price: 12.5, category: 'Fruits'},
    {name:'Pear', price: 8.5, category: 'Fruits'},
])
db.products.find()

// IF - ELSE
// Vamos usar IF - ELSE pra criar um novo campo
// Usar este novo campo para filtrar resultdos. 

// select *, GETDATE() as date from products
db.products.aggregate([
    {
        $addFields: {
          date: new Date(),
          tag: {
            $cond:{
                if: {$gte:['$price',10]},
                then: 'Caro',
                else: 'Barato'
            }
          }
        }
    },
    {$match: {
      tag:'Caro'
    }}
])

db.products.aggregate([
    {
        $set: {
          date: new Date(),
          tag: {
            $cond:{
                if: {$gte:['$price',10]},
                then: 'Caro',
                else: 'Barato'
            }
          }
        }
    },
    {$match: {
      tag:'Caro'
    }}
])

// Atalho para fazer If-ELSE
db.products.aggregate([
    {
        $set: {
          date: new Date(),
          tag: {
            $cond:[{$gte: ['$price',10]}, 'Caro', 'Barato']
          }
        }
    },
    {$match: {
      tag:'Caro'
    }}
])

// group by
/*
select category, count(1) from products group by category
*/
db.products.aggregate([
    {
        $group: {
          _id: '$category',
         qty: {$sum: 1},
         priceTotal: {$sum: '$price'},
         priceAverage: {$avg: '$price'}
        }
    }
])

//bucket 
db.products.aggregate([
  {
    $bucket: {
      groupBy: '$price',
      boundaries: [ 0, 5, 10, 15, 20, 25 ],
      default: '0',
      output:{
        count:{$sum:1},
        product:{$push:{name:'$name',price:'$price'}},
      }
    }    
  },
  { $sort: {count:-1}}
])

db.products.aggregate([
  {
    $bucketAuto: {
      groupBy: '$price',
      buckets: 6,
      output: {
        count:{$sum:1},
        product:{$push:{name:'$name',price:'$price'}},
      }
    }
  }
])

// $FACET
db.products.aggregate([
  {
    $facet: {
      byCategories:[
        {$group:{_id:'$category', qty:{$sum:1}}}
      ],
      byPrice:[      
        {
          $bucketAuto: {
            groupBy: '$price',
            buckets: 6,
            output: {
              count:{$sum:1},
              product:{$push:{name:'$name',price:'$price'}},
            }
          }
        }
      ]
    }
  }
])

// UNWIND
db.movie.insertMany([
  {title:'IT', tags: ['Horror']},
  {title:'Megan', tags: ['Horror', 'SCI-FI', 'Triller']},
  {title:'Avatar', tags: ['Action', 'Adventure', 'Fantasy']},
  {title:'Andor', tags: ['Action', 'Adventure', 'Drama']},
  {title:'Barbie', tags: ['Comedy', 'Adventure', 'Fantasy']},
])


db.movie.aggregate([
  {$unwind: '$tags'},
  {$group: {
    _id: '$tags',
    qty: {
      $sum: 1
    }
  }},
  { $sort: {qty:-1}},
  { $limit:3}
])


// Substring
db.movie.aggregate([
  {$unwind: '$tags'},
  {
    $set: {
      letter: {$substr: ['$title',0,1]}
    }
  },
  {
    $group: {
      _id: '$letter',
      movies: {
        $addToSet: { title:'$title'}
      }
    }
  },
  {$sort: {_id: 1}}
])

db.movie.aggregate([
  {$unwind: '$tags'},
  {
    $set: {
      letter: {$substr: ['$title',0,1]}
    }
  },
  {
    $group: {
      _id: '$letter',
      movies: {
       $push:'$$ROOT'
      }
    }
  },
  {$sort: {_id: 1}}
])

// MAP
db.movie.aggregate([
  {
    $set: {
      tags: {
        $map:{
          input:'$tags',
          as: 'nome_de_variavel_qualquer',
          in:{ $toUpper: '$$nome_de_variavel_qualquer'}
        }
      }
    }
  }
])

// REDUCE
/**
 * const tags = [
 *    "HORROR",
 *    "SCI-FI",
 *    "TRILLER"
 *  ]
 * 
 * Resultado esperado após aplicação do reduce = const output = "HORROR, SCI-FI,TRILLER"
 * 
 * Javascript ser algo como const output = tags.reduce((v,t) => v + t + ',', ' ')
 * 
 * looping 1: '' + HORROR + ,                 = 'HORROR,'
 * looping 2: 'HORROR,' + 'SCI-FI' + ','      = 'HORROR,SCI-FI,'
 * looping 2: 'HORROR,SCI-FI' + 'TRILLER' ',' = 'HORROR,SCI-FI,TRILLER,'
 */
db.movie.aggregate([
  {
    $set: {
      tags: {
        $reduce:{
          input:'$tags',
          initialValue: '',
          in: { $concat: ['$$value', '$$this', ',']}
        }
      }
    }
  }
])


db.movie.aggregate([
  {
    $set: {
      tags: {
        $map:{
          input:'$tags',
          as: 'nome_de_variavel_qualquer',
          in:{ $toUpper: '$$nome_de_variavel_qualquer'}
        }
      }
    }
  },
  {
    $set: {
      tags: {
        $reduce:{
          input:'$tags',
          initialValue: '',
          in: { $concat: ['$$value', '$$this', ',']}
        }
      }
    }
  } 
])

// Sample
// Peg itens aleatórios de uma collection
db.movie.aggregate([
  {
    $sample: {
      size: 2
    }
  } 
])




















































