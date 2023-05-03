# FastApiSessionToAsyncSession
Migrate From Session To AsyncSession


```python
from fastapi import FastAPI, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload, sessionmaker

from models import Base, User
from database import engine

app = FastAPI()

SessionLocal = sessionmaker(bind=engine)
async_session = sessionmaker(bind=engine, class_=AsyncSession)


async def get_db():
    async with async_session() as session:
        async with session.begin():
            yield session


# Note: "get_db" instead of "get_db()" because "yield" makes "get_db()" a generator.
@app.get("/users/{user_id}")
async def read_user(user_id: int, db: AsyncSession = Depends(get_db)):
    statement = selectinload(User.email).where(User.id == user_id)
    result = await db.execute(statement)
    user = result.scalar()
    return user


async def create_users(db: AsyncSession):
    new_user = User(name="username")
    db.add(new_user)
    await db.commit()
    await db.refresh(new_user)
    return new_user

@app.post("/users/")
async def create_user(db: AsyncSession = Depends(get_db)):
    user = await create_users(db)
    return user
```




