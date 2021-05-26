Results
=========

以下是实验代码（修改部分已添加注释）和运行截图。（生成的db在附件2中）

model.py 

.. code:: python

    # Software Architecture and Design Patterns -- Lab 2 starter code
    # Copyright (C) 2021 Hui Lan

    from dataclasses import dataclass

    @dataclass
    class Article:
        article_id:int
        text:str
        source:str
        date:str
        level:int
        question:str
        

    class NewWord:
        def __init__(self, username, word='', date='yyyy-mm-dd'):
            self.username = username
            self.word = word
            self.date = date

    class User:
        def __init__(self, username, password='12345', start_date='2021-05-19', expiry_date='2031-05-19'):
            self.username = username
            self.password = password
            self.start_date = start_date
            self.expiry_date = expiry_date
            self._read = []

        def read_article(self, article):
            self._read.append(article)  # <------ 修改

orm.py

.. code:: python

    # Software Architecture and Design Patterns -- Lab 2 starter code
    # Copyright (C) 2021 Hui Lan

    from sqlalchemy import Table, MetaData, Column, Integer, String, Date, ForeignKey
    from sqlalchemy.orm import mapper, relationship

    import model

    metadata = MetaData()

    articles = Table(
        'articles',
        metadata,
        Column('article_id', Integer, primary_key=True, autoincrement=True),
        Column('text', String(10000)),
        Column('source', String(100)),
        Column('date', String(10)),
        Column('level', Integer, nullable=False),
        Column('question', String(1000)),    
        )

    readings = Table(                                           # <------ 修改
        'readings',
        metadata,
        Column('id', Integer, primary_key=True, autoincrement=True),
        Column('username', String(100), ForeignKey('users.username')),
        Column('article_id', Integer, ForeignKey('articles.article_id')),
        )

    users = Table(
        'users',
        metadata,
        Column('username', String(100), primary_key=True),
        Column('password', String(64)),
        Column('start_date', String(10), nullable=False),
        Column('expiry_date', String(10), nullable=False),  
        )

    newwords = Table(
        'newwords',
        metadata,
        Column('word_id', Integer, primary_key=True, autoincrement=True),
        Column('username', String(100), ForeignKey('users.username')),
        Column('word', String(20)),
        Column('date', String(10)),
        )


    def start_mappers():                                            # <------ 修改
        articles_mapper = mapper(model.Article, articles)
        newwords_mapper = mapper(model.NewWord, newwords)
        users_mapper = mapper(model.User, users, 
            properties = {
                "newwords": relationship(newwords_mapper),
                "_read": relationship(
                    articles_mapper,
                    secondary = readings,
                    collection_class = list,
                ),
            }
        )

.. figure:: result.png
    
    result

