# KoboldAI Horde

This turns KoboldAI into a giant crowdsourced distributed cluster. It allows people without a powerful GPU to use KAI by relying on spare/idle resources provided by the community.
It also allows clients other than KAI, such as games and apps, to use KAI-provided generations.

You can find visit [official KoboldAI Horde](https://koboldai.net)

# Registering

To use the horde you need to have a registered amount, or use anonymous mode.

To register an account, go to https://koboldai.net/register and login with one of the available services. Once you do you'll be redirected back to the main page, so go back to /register and you'll see a form where you can put a username. Add one in and it will automatically store a user object for you and provide an API key to identify you. 
Store this API key and use it for your client or bridge.
By logging in first, you can change your username and API key at any time. 
Be aware that the account is unique per service, so even if you use the same email for discord and google, your user ID will be different for each!
We don't store any identifiable information other than the ID string sent by the oauth for your user. We only use this for user uniqueness, and no other purpose.

If you want, you can also create a pseudonymous account, without logging in with oauth. However *we will not maintain such accounts*. If you lose access to it, you'll have to make a new one. If someone copies your API Key, they can impersonate. You cannot change the username or API key anymore etc. If you don't want these risks, login to one of the available services instead.

If you do not want to login even with a pseudonymous account, you can use this service anonymously by using '0000000000' as you API key. However your usage and contributions will be not be tracked. Be aware that if this service gets too overloaded, anonymous mode might be turned off!

# Generating Prompts

To request the generation for a prompt, you need to send a post request like so:

Synchronously
```
curl -H "Content-Type: application/json" -d '{"prompt":"I entered into an argument with a clown", "params":{"max_length":16, "frmttriminc": true, "n":2}, "api_jey":"0000000000", "models":["KoboldAI/fairseq-dense-13B-Nerys-v2"]}' https://koboldai.net/api/latest/generate/sync
```
Asynchronously
```
curl -H "Content-Type: application/json" -d '{"prompt":"I entered into an argument with a clown", "params":{"max_length":16, "frmttriminc": true, "n":2}, "api_jey":"0000000000", "models":["KoboldAI/fairseq-dense-13B-Nerys-v2"]}' https://koboldai.net/api/latest/generate/async
```

The "params" dictionary is the same as the parameters you pass to the KoboldAI API in the `api/latest/generate` endpoint, the only difference is that the "prompt" is outside the "params" dictionary.

With one important difference, the "Gens per action" param `n` can be as high as you want! Each server will only handle 1 at a time, but multiple server will be able to work on your request at the same time.

Pass an API Keyin order to track your usage.

Pass the desired model names in the "model" arg to allow only KAIs running one of those models to fulfil your request. If you skip the "models" arg, all KAI instances will be able to generate for you, but of course the result will vary per model.

The `max_length` and `max_content_length` params that you pass are your wish. If the KAI server checking your request has a lower limit, they will skip all generation requests with a higher wish

In Synchronous mode, this will return a list of prompts. So you can immediately use them. However if you connection is disrupted you will lose your generation, as you won't know your UUID

In Asynchronous mode  This request will return a UUID
```
{"id": "34a9f91a-6db5-4d4c-962f-c4d795739610"}
```

You can now request the status of this prompt using this UUID from the server

```
curl https://koboldai.net/api/latest/generate/prompt/2a72f411-a4c3-49e1-aad4-41005e1ff769
{"finished": 1, "processing": 0, "waiting": 1, "done": false, "generations": [" as he stood before me."]}
```

Once the `finished` arg is equal to your `n`, then your request is completed.

## Specifying servers

You can optionally specify only specific servers to generate for you. Grab one or more server IDs from `/servers` and then send it with your payload as a list in the "servers" arg. Your generation will only be fulfiled by servers with the specified IDs

## Specifying softprompts

You can optionally specify only specific softprompts to use with generation. You can type all or part of the softprompt **filename** that you want to be used. Only servers which can find that softprompt name for their model will fulfil your request. If you don't specify anything, or specify an empty list, then any active softprompts will be turned off for your generation.

The softprompts you specify need to exist on the KAI server side. If you want to be able to generate using a specific softprompt on other servers, distribute it and ask people to add it to their softprompts folder.

# Using KoboldAI client

**You KoboldAI client must be using the UNITED branch!**

The United branch of KoboldAI now supports using the bridge directly from its interface. To use it go to the AI menu on the top left, then select Online Services > KoboldAI Horde. In the next window that opens, you have to fill in the url of the Horde (You can use `https://koboldai.net`) and then type a username. If the models window hasn't appeared yet, click away from the textbox and it should. You can select one of more models that you want to use for your generations, or All, if you don't care. Then finally click `Load`.

![](gui_select.png)

You can also start KAI directly in horde mode by using the command line in the `play.(sh|bat)` file. Pass the arguments to start a KAI instance in cluster mode like so (Change "0000000000" to your own API KEY, if you have one.)

LINUX

```bash
APIKEY=0000000000
play.sh --path https://koboldai.net --model CLUSTER --apikey ${APIKEY}
```

WINDOWS


```bash
play.bat --path https://koboldai.net --model CLUSTER --apikey 0000000000
```

This will use any available model on the cluster. If you want to use only specific models, pass the wanted modules via one or more `req_model` args. Example `--req_model "KoboldAI/fairseq-dense-13B-Nerys-v2" --req_model "KoboldAI/fairseq-dense-2.7B-Nerys"`

Once the KAI starts in cluster mode, any request will be sent to the cluster

# Joining the horde

This repository comes with a little bridge script which you can run on your own machine (windows or linux). It will take care of communicating between KAI cluster server and your own KAI. This will allow people to use their own PCs to support the KAI cluster.

**You KoboldAI instance must be using the UNITED branch!**

## Android/Termux

Through Termux, you can run the bridge on your phone and connect it to a KAI Colab instance.

* [Install Termux from F-Droid](https://f-droid.org/en/packages/com.termux/) (Install F-Droid if you haven't already)
* Open termux and type this script 
   ```bash
   curl https://raw.githubusercontent.com/db0/KoboldAI-Horde/master/termux_bridge_setup.sh | sh
   ```
* If the script fails, try to rerun it. Sometimes it needs two runs
* cd to the new git directory `cd KoboldAI-Horde` if you're not in it already
* You can start the bridge on two ways. 
   * Edit clientData.py and add your KoboldAI Worker URL and your API key, then start the bridge with `python bridge.py -vv`
   * Put the variables in the command line directly `python bridge.py -k https://YOUR_KAI_WORKER_URL_HERE -a YOUR_API_KEY_HERE -n "My Awesome Instance" -vv`
* You can also combine the two, where you fill in your `clientData.py` with your API key and worker name, and then pass the URL on the command line if it changes every time.

You should now be seeing logs from the bridge, showing you the status of its connection
   


## Windows/Linux

* First clone this repo and then open a terminal/console and `cd` into it
* Make sure you have python3 installed
* install the requirements with pip: `python -m pip install -r bridge_requirements.txt --user`
* Copy clientData_template.py into clientData.py
* Edit the clientData.py file and add your own username and password. The password is stored in plaintext for now, so don't reuse an existing one.
* Edit the clientData.py file and add your KAI server. If it's a local instance, leave it as it is. If it's a remote instance, fill in the URL and port accordingly.
* Modify your KAI settings from the GUI so that the "Amount to Generate" and "Typical Sampling" are at the max values your KAI instance can handle. This doesn't mean all requests will use this amount. It just limits which requests your server will choose to fulfil.
* Finally, run the script: `python bridge.py`
   * Optionally, provide bridge arguments via command line. The args on the command line will override clientData.py vars, so you can use this to run multiple bridges from the same location. See `python bridge.py -h`

If all goes well, it will connect to your KAI instance and then will start polling the cluster for incoming requests.

A server will be considered "stale" and not shown in the general list, if it doesn't check in for at least 5 minutes. You can see still them through their individual UUID endpoint, and it will continue where it left off as soon as it checks back in to fulfil requests. In fact, it doesn't technically need to be the same server. You can switch to a different box and long as your server name and auth is the same, your stats will carry on. This means that you can keep plugging in different collab instances while retaining the same stats.

## Softprompts

The bridge will automatically enable or disable softprompts at the clients request, assuming they exist in your files. If you want to help more specialized requests, make sure you download and install other people's softprompts on your server so that they are available to generate for people using them.

# Other endpoints

* GET `/api/latest/servers` To see the info of all currently active servers and their statistics.
   ```json
   [{"name": "Db0's Awesome Instance", "id": "019e34e3-3109-4ea9-820a-b0c4f6a53c07", "model": "KoboldAI/fairseq-dense-2.7B-Nerys", "max_length": 80, "max_content_length": 1632, "tokens_generated": 30, "requests_fulfilled": 2, "latest_performance": "1.36 tokens per second"}]
* GET `/api/latest/servers/<UUID>` To see the information of a specific server by UUID.
* GET `/api/latest/users` To see the info of all registered users and their statistics
* GET `/api/latest/user/<ID>` To see the info of a specific user. ID is the integer at the end of their name.
* GET `/api/latest/models` Which models are currently active and how many servers are running each
* GET `/api/latest/kudos/transfer` Transfer Kudos to another user

## Other Info

The cluster does not save any prompts nor generations locally. It's all stored in memory. Furthermore, requested prompts and their generations are wiped after 10 minutes of inactivity

The bridge also does not save any prompts, but of course this is not under my control as the bridges run locally on the various cluster nodes. As such, you should not prompt with anything you do not want others to see!

This system stores how many tokens you requested to generate, and how many your own servers have generated for others. This is not used yet, but eventually this will be how we balance resources among the users.

## Advanced Usage: Local + Horde KAI

If you want to both play with KAI AND share resources with the community, you can achieve this by running two instances of KAI side by side. One normal one, and one in cluster mode. That way when you're using the clustered KAI, you will ensure there's always at least one instance to serve you (your own), while also taking advantage of any other instances that are onboarded at the time.

1. start KAI as you would normally do with `play.(sh|bat)` and load the model you want.
2. open a terminal window at your KAI installation, and now run `play.(sh|bat)` using a different port in CLUSTER mode. This will start open another KAI window, while leaving your model-loaded KAI intact. Make sure the `req_model` you pass, includes the model loaded in your own KAI instance.

```bash
play.bat --port 5002 --path https://koboldai.net --model CLUSTER --apikey 0000000000 --req_model "KoboldAI/fairseq-dense-13B-Nerys-v2" --req_model "KoboldAI/fairseq-dense-2.7B-Nerys"
```

Now use the CLUSTER KAI to play. You will see the requests being fulfilled by the model-loaded KAI after you press the button.