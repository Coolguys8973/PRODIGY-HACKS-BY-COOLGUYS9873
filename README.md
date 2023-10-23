import { h } from "preact"
import { getPlayer, getWorld, saveGame, setFromUserID } from "../hack"
import { InputTypes, success, confirm, error, customMessage } from "../swal"
import { Player } from "../types/player"
import { Category } from "./base/categories"
import { withCategory } from "./base/registry"

const keyboardEvent = (event: KeyboardEvent) => {
    const { playerContainer } = getPlayer() as Player
    let { x, y } = playerContainer
    switch (event.key) {
    case "ArrowUp":
        y -= 10
        break
    case "w":
        y -= 10
        break
    case "ArrowDown":
        y += 10
        break
    case "s":
        y += 10
        break
    case "ArrowLeft":
        x -= 10
        break
    case "a":
        x -= 10
        break
    case "ArrowRight":
        x += 10
        break
    case "d":
        x += 10
        break
    }

    playerContainer.locomotion.onMovePlayer(x, y)
}

withCategory(Category.UTILITY, ({ hack, toggle }) => {
    hack("Save Character", "Use this to make sure your character is saved.", async () => {
        saveGame() // TODO: If on extension, use _ method.
        success("Your character has been saved.")
    })
    let teleportingInterval
    toggle("Toggle Click Teleporting", (hack, player, gameData, toggled) => {
        if (toggled) {
            teleportingInterval = setInterval(() => {
                player._playerContainer.walkSpeed = 500
            }, 100)
            success("You will now teleport when you click the screen.")
        } else {
            clearInterval(teleportingInterval)
            player._playerContainer.walkSpeed = 1.5
            success("You will no longer teleport when you click the screen.")
        }
    })
    hack("Edit Walkspeed", "Edit the walkspeed of your character.", async (hack, player) => {
        const walkSpeed = await InputTypes.float("Please enter the walkspeed you want your character to be at. The default is 1.5")
        if (player._playerContainer?.walkSpeed) {
            player._playerContainer.walkSpeed = walkSpeed || 1.5
        } else {
            const interval = setInterval(() => {
                if (player._playerContainer?.walkSpeed) {
                    player._playerContainer.walkSpeed = walkSpeed || 1.5
                    clearInterval(interval)
                }
            }, 100)
        }
        success(`You are now at a walkspeed of ${walkSpeed}.`)
    })
    hack("Reset Account", "Completely resets your account.", async (hack, player) => {
        const confirmed = await confirm("Are you sure you want to reset your account?")
        if (confirmed) {
            player.resetAccount()
            success("Your account has been reset.")
        } else {
            error("Cancelled by user.")
        }
    })
    hack("Find the UserId of People on the Screen", "Get's the UserId of every player on the screen currently", async (hack) => {
        const users = hack?._state?._current?.playerList ?? {}
        if (Object.keys(users).length === 0) {
            error("There are no other players on the screen.")
        } else {
            customMessage({
                title: "The user's on the screen are:",
                html: <div>
                    <ul className="list-disc list-inside">
                        {Object.keys(users).map(user => <li key={user}>
                            User Id: {user} - {users[user].nameText.textSource.source}
                        </li>)}
                    </ul>
                </div>
            })
        }
    })
    hack("Duplicate an Account", "Copy an account from it's UserId.", async () => {
        const userId = await InputTypes.string("Please enter the UserId of the account you want to copy. Warning: this will delete your current account.")
        const account = await setFromUserID(userId) // TODO: If on extension, use _ method.
        if (account) {
            success(`Account ${userId} has been copied to your account.`)
        } else {
            error(`Account ${userId} could not be copied.`)
        }
    })
    hack("Close All Popups", "Closes every popup. Warning: If you close a 418 popup, your account will not save until you reload.", async () => {
        _.instance.prodigy.open.menuCloseAll()
        success("All open popups were closed.")
    }, true)
    hack("Generate Alt Account", "This will generate an alternative account from your data.", async (hack, player) => {
        const [username, password] = await (await fetch(`https://prodigy-api.hostedposted.com/generate-account?${
            new URLSearchParams({
                username: await InputTypes.string("Please enter your username. This will not be your actual username. The actual one will be this plus the last initial you choose, plus a number which corresponds to the number of accounts that have started with the username so far."),
                password: await InputTypes.string("Please enter your password"),
                lastInitial: await InputTypes.string("Please enter your last initial")
            })
        }`)).json()

        const basicAuth = `Basic ${btoa(`${username}:${password}`)}`

        const { token, userID } = await (await fetch("https://prodigy-api.hostedposted.com/token", {
            headers: {
                Authorization: basicAuth
            }
        })).json()

        const saveInfo = await fetch(`https://api.prodigygame.com/game-api/v3/characters/${userID}`, {
            headers: {
                accept: "*/*",
                "accept-language": "en-US,en;q=0.9",
                authorization: `Bearer ${token}`,
                "content-type": "application/json",
                "sec-ch-ua": "\".Prodigy/A)Brand\";v=\"99\", \"Google Chrome\";v=\"103\", \"Chromium\";v=\"103\"",
                "sec-ch-ua-mobile": "?0",
                "sec-ch-ua-platform": "\"Windows\"",
                "sec-fetch-dest": "empty",
                "sec-fetch-mode": "cors",
                "sec-fetch-site": "same-site",
                Referer: "https://math.prodigygame.com/",
                "Referrer-Policy": "strict-origin-when-cross-origin"
            },
            body: JSON.stringify({
                data: JSON.stringify(player.getUpdatedData(true)),
                userID
            }),
            method: "POST"
        })

        if (!saveInfo.ok) {
            error(`Could not save data to generated account ('${username}' '${password}'). ${saveInfo.statusText} ${saveInfo.status}. ${await saveInfo.text()}`)
            return
        }

        if (username && password) {
            success(`Account '${username}' has been generated with a password of '${password}'.`)
        }
    }, true)
    toggle("Toggle Arrow Key Movement", (hack, player, gameData, toggled) => {
        if (toggled) {
            window.addEventListener("keydown", keyboardEvent)
            success("You will now be able to move your character with the arrow keys or WASD.")
        } else {
            window.removeEventListener("keydown", keyboardEvent)
            success("You will no longer be able to move your character with the arrow keys or WASD.")
        }
    }, () => false)
    hack("Teleport To Map", "Easily teleport to anywhere on the map.", async () => {
        const world = getWorld()
        const locationNames = Object.values(world.zones).map((zone: any) => zone.name)
        const locationIds = Object.keys(world.zones)

        const location = await InputTypes.select("Please select the location you want to teleport to.", locationNames)
        const locationId = locationIds[location]
        const locationName = locationNames[location]

        const zone = world.zones[locationId]
        const maps = Object.keys(zone.maps)

        const map = await InputTypes.select("Please select the map you want to teleport to.", maps)

        zone.teleport(maps[map], 500, 500, {}, {})

        success(`You have been teleported to ${locationName} - ${maps[map]}.`)
    })
    hack("Skip Tutorial", "Skip's the intro tutorial for new prodigy accounts.", async (hack, player) => {
        const setQuest = (t: string, i: number, n?: unknown, e?: unknown) => {
            _.instance.prodigy.world.getZone(t).testQuest(i, n, e)
            try {
                Object.fromEntries(hack.state.states).TileScreen.process()
            } catch {}
        }

        setQuest("house", 2)
        setQuest("academy", 2)
        player.state.set("tutorial-0", 4)
        // @ts-ignore
        player.backpack.addKeyItem(13, 0)
        player.tutorial.data.menus[14] = [1]
        _.instance.prodigy.open.map(true, [])
        player.onTutorialComplete()
        player.data.level = Math.max(_.player.data.level, 5)
    }, true)
    hack("Go To Prodigy Wiki", "Opens up a new window with the prodigy wiki.", async () => {
        window.open("https://prodigywiki.com")
    })
})
