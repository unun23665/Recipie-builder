import React, { useState, useEffect } from 'react';

interface Ingredient {
  name: string;
  quantity: number;
  unit: string;
}

interface Recipe {
  name: string;
  ingredients: Ingredient[];
  instructions: string;
  preparationTime: number;
}

const RecipeForm = () => {
  const [recipe, setRecipe] = useState<Recipe>({
    name: '',
    ingredients: [],
    instructions: '',
    preparationTime: 0,
  });

  const [errors, setErrors] = useState({
    name: '',
    ingredients: [],
    instructions: '',
    preparationTime: '',
  });

  const units = ['grams', 'cups', 'tablespoons', 'teaspoons'];

  const handleAddIngredient = () => {
    setRecipe((prevRecipe) => ({
      ...prevRecipe,
      ingredients: [...prevRecipe.ingredients, { name: '', quantity: 0, unit: units[0] }],
    }));
  };

  const handleRemoveIngredient = (index: number) => {
    setRecipe((prevRecipe) => ({
      ...prevRecipe,
      ingredients: prevRecipe.ingredients.filter((_, i) => i !== index),
    }));
  };

  const handleIngredientChange = (index: number, field: 'name' | 'quantity' | 'unit', value: string | number) => {
    setRecipe((prevRecipe) => ({
      ...prevRecipe,
      ingredients: prevRecipe.ingredients.map((ingredient, i) => {
        if (i === index) {
          return { ...ingredient, [field]: value };
        }
        return ingredient;
      }),
    }));
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    const validationErrors = {
      name: '',
      ingredients: [],
      instructions: '',
      preparationTime: '',
    };

    if (recipe.name.length < 3) {
      validationErrors.name = 'Recipe name should be at least 3 characters';
    }

    recipe.ingredients.forEach((ingredient, index) => {
      if (ingredient.name.length < 2) {
        validationErrors.ingredients[index] = 'Ingredient name should be at least 2 characters';
      }
      if (ingredient.quantity <= 0) {
        validationErrors.ingredients[index] = 'Ingredient quantity should be greater than 0';
      }
    });

    if (recipe.preparationTime <= 0) {
      validationErrors.preparationTime = 'Preparation time should be a positive number';
    }

    setErrors(validationErrors);

    if (!Object.values(validationErrors).some((error) => error)) {
      // Display recipe summary
    }
  };

  const handleSaveRecipe = () => {
    const storedRecipes = localStorage.getItem('recipes');
    if (storedRecipes) {
      const parsedRecipes = JSON.parse(storedRecipes);
      parsedRecipes.push(recipe);
      localStorage.setItem('recipes', JSON.stringify(parsedRecipes));
    } else {
      localStorage.setItem('recipes', JSON.stringify([recipe]));
    }
  };

  const calculateTotalQuantity = (unit: string) => {
    return recipe.ingredients.reduce((acc, ingredient) => {
      if (ingredient.unit === unit) {
        return acc + ingredient.quantity;
      }
      return acc;
    }, 0);
  };

  return (
    <div className="max-w-3xl mx-auto p-4">
      <form onSubmit={handleSubmit}>
        <div className="mb-4">
          <label className="block text-gray-700 text-sm font-bold mb-2" htmlFor="recipe-name">
            Recipe Name
          </label>
          <input
            className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            id="recipe-name"
            type="text"
            value={recipe.name}
            onChange={(event) => setRecipe((prevRecipe) => ({ ...prevRecipe, name: event.target.value }))}
          />
          {errors.name && <p className="text-red-500 text-xs italic">{errors.name}</p>}
        </div>

        <div className="mb-4">
          <label className="block text-gray-700 text-sm font-bold mb-2" htmlFor="ingredients">
            Ingredients
          </label>
          {recipe.ingredients.map((ingredient, index) => (
            <div key={index} className="flex items-center mb-2">
              <input
                className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                value={ingredient.name}
                onChange={(event) => handleIngredientChange(index, 'name', event.target.value)}
              />
              <input
                className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="number"
                value={ingredient.quantity}
                onChange={(event) => handleIngredientChange(index, 'quantity', Number(event.target.value))}
              />
              <select
                className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                value={ingredient.unit}
                onChange={(event) => handleIngredientChange(index, 'unit', event.target.value)}
              >
                {units.map((unit) => (
                  <option key={unit} value={unit}>
                    {unit}
                  </option>
                ))}
              </select>
              <button
                className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
                type="button"
                onClick={() => handleRemoveIngredient(index)}
              >
                Remove
              </button>
              {errors.ingredients[index] && <p className="text-red-500 text-xs italic">{errors.ingredients[index]}</p>}
            </div>
          ))}
          <button
            className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
            type="button"
            onClick={handleAddIngredient}
          >
            Add Ingredient
          </button>
        </div>

        <div className="mb-4">
          <label className="block text-gray-700 text-sm font-bold mb-2" htmlFor="instructions">
            Instructions
          </label>
          <textarea
            className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            id="instructions"
            value={recipe.instructions}
            onChange={(event) => setRecipe((prevRecipe) => ({ ...prevRecipe, instructions: event.target.value }))}
          />
        </div>

        <div className="mb-4">
          <label className="block text-gray-700 text-sm font-bold mb-2" htmlFor="preparation-time">
            Preparation Time
          </label>
          <input
            className="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            id="preparation-time"
            type="number"
            value={recipe.preparationTime}
            onChange={(event) => setRecipe((prevRecipe) => ({ ...prevRecipe, preparationTime: Number(event.target.value) }))}
          />
          {errors.preparationTime && <p className="text-red-500 text-xs italic">{errors.preparationTime}</p>}
        </div>

        <button
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
          type="submit"
        >
          Submit
        </button>
        <button
          className="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
          type="button"
          onClick={handleSaveRecipe}
        >
          Save Recipe
        </button>
      </form>

      <div className="mt-4">
        <h2 className="text-lg font-bold">Total Quantity</h2>
        {units.map((unit) => (
          <p key={unit}>{unit}: {calculateTotalQuantity(unit)}</p>
        ))}
      </div>
    </div>
  );
};

export default RecipeForm;
