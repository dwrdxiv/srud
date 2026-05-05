<template>
    <div class="links">
            <RouterLink to="/">Home</RouterLink>
            <RouterLink to="/inventario">Inventory</RouterLink>

    </div>
</template>

<style scoped>
.links {
    height: 2.2rem;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 2px;
    background-color: #dbdfe7;
}
.links a {
    color: rgb(0, 0, 0);
    text-decoration: none;
    font-family: Arial, "Helvetica Neue", Helvetica, sans-serif;
    background-color: white;
    height: 100%;
    min-width: 6rem;
    text-align: center;
    line-height: 2.2rem;
    transition: background-color 0.3s ease-in-out, transform 0.2s ease-in-out, color 0.3s ease-in-out;
}
.links a:hover {
    background-color: rgb(0, 0, 0);
    color: rgb(255, 255, 255);
}

.links a:active {
    transform: scale(0.95);
}

</style>