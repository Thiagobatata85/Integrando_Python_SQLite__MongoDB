# Integrando_Python_SQLite__MongoDB
Integrando

from sqlalchemy.orm import declarative_base
from sqlalchemy.orm import Session
from sqlalchemy import Column
from sqlalchemy import create_engine
from sqlalchemy import select
from sqlalchemy import Integer
from sqlalchemy import String
from sqlalchemy import Float
from sqlalchemy import ForeignKey

#Declaração das Classes para o Modelo ORM
Base = declarative_base()

class Client(Base):
    """
    Esta classe representa a tabela cleinte do SQlite.
    """
    __tablename__ = "client"
    #Atributos
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    cpf = Column(String(9))
    address = Column(String(30))


    def __repr__(self):
        return f"Client(id={self.id}, name={self.name}, address={self.address})"

class Account(Base):
    """
        Esta classe representa a tabela account do SQlite.
        """
    __tablename__ = "account"
    # Atributos
    id = Column(Integer, primary_key=True)
    tipo = Column(String(2))
    agency = Column(Integer)
    number = Column(Integer)
    balance = Column(Float)
    client_id = Column(Integer,ForeignKey("client.id"), nullable=False)

    def __repr__(self):
         return f"Account(id={self.id}, tipo={self.tipo}, saldo={self.balance})"


    #Realiza a conexão com o banco de dados
engine = create_engine("sqlite://")

#Cria as classes como tabelas no banco de dados
Base.metadata.create_all(engine)

#Faz a persistencia as Informações no Banco de Dados SQlite

with Session(engine) as session:
    thiago = Client(name="Thiago Mateus",
                    cpf="360.235.656.65",
                    address=" Rua 24 de Abril, 61"
                    )

    poli = Client(name="Poliana Brezolin",
                    cpf="365.653.654.35",
                    address=" Rua 23 de Março, 123"
                    )
    pietro = Client(name=" Pietro Santos",
                    cpf="654.653.895.65",
                    address=" Rua 12,56"
                    )

    account1=Account(client_id="1",
                     tipo="cc",
                     agency=3001,
                     number=364565,
                     balance=45000
                     )
    account2 = Account(client_id="2",
                       tipo="cc",
                       agency=3502,
                       number=13363,
                       balance=12000
                       )
    account3 = Account(client_id="3",
                       tipo="cc",
                       agency=9003,
                       number=793664,
                       balance=56300
                       )

    #Envia as informações para o Bnaco de Ddos(presitencia de dados)
    session.add_all([thiago,poli,pietro])
    session.add_all([account1, account2,account3])
    session.commit()


    #Consulta as informações salvas no bando de dados SQlite

    print('Recuperando clientes a partir de uma condição de filtragem:')
    stmt_clients = select(Client).where(Client.name.in_(['Piangelo Diatore', 'Pietro de Lorenzo']))
    for result in session.scalars(stmt_clients):
        print(result)

    print("\nRecuperando clientes de maneira ordenada:")
    stmt_order = select(Client).order_by(Client.name.desc())
    for result in session.scalars(stmt_order):
        print(result)

    print("\nRecuperando contas de maneira ordenada:")
    stmt_accounts = select(Account).order_by(Account.tipo.desc())
    for result in session.scalars(stmt_accounts):
        print(result)

    print("\nRecuperando contas e clientes:")
    stmt_join = select(Client.name, Account.tipo, Account.balance).join_from(Client, Account)
    connection = engine.connect()
    results = connection.execute(stmt_join).fetchall()
    for result in results:
        print(result)

    session.close()
