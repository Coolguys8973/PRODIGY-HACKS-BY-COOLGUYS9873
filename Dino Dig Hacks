import { success, error, InputTypes } from "../swal"
import { Category } from "./base/categories"
import { withCategory } from "./base/registry"

withCategory(Category.MINIGAME, ({ hack }) => {
    hack("Edit Dino Dig Walkspeed", "Edit the walkspeed of your player in dino dig.", async (hack) => {
        const value = await InputTypes.integer("Please enter the walkspeed you want your character to be at. The default is 1.5", 1, 100)
        hack._state.states.get("DinoDig").walkSpeed = value
        success(`You are now at a walkspeed of ${value}.`)
    })
    hack("Extra Time In Dino Dig", "Add 100 days to the timer in dino dig.", async (hack) => {
        if (!hack._state.states.get("DinoDig")?.timer?.addTime) {
            error("You are not in dino dig.")
            return
        }
        hack._state.states.get("DinoDig")?.timer?.addTime(8.64e9)
        success("Added 100 days to the timer.")
    })
    hack("End Dino Dig", "End the current time in dino dig.", async (hack) => {
        if (!hack._state.states.get("DinoDig")?.timer?.addTime) {
            error("You are not in dino dig.")
            return
        }
        hack._state.states.get("DinoDig")?.timer?.addTime(-9e15) // Subtract 285,388.1278538812694 years.
        success("The dino dig game has been ended.")
    })
})

alert("MADE BY COOLGUYS9873 MY USERNAME IN PRODIGY IS KING OF THE FOREST")
