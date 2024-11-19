int):
    """Supprime entre 1 et 100 messages dans le salon actuel."""
    if 1 <= nombre <= 100:
        await ctx.channel.purge(limit=nombre + 1)
        await ctx.send(f" {nombre} messages ont été supprimés !", delete_after=5)
    else:
        await ctx.send("Vous devez spécifier un nombre entre 1 et 100.", delete_after=5)

@bot.command(name='ban')
@commands.has_permissions(ban_members=True)
async def ban(ctx, member: discord.Member, *, reason="Aucune raison spécifiée"):
    await member.ban(reason=reason)
    await ctx.send(f"{member} a été banni pour : {reason}")

# Débannir un utilisateur
@bot.command(name='unban')
@commands.has_permissions(ban_members=True)
async def unban(ctx, user: discord.User):
    await ctx.guild.unban(user)
    await ctx.send(f"{user} a été débanni.")

# Mute temporaire
@bot.command(name='mute')
@commands.has_permissions(manage_roles=True)
async def mute(ctx, member: discord.Member, duration: int, *, reason="Aucune raison spécifiée"):
    role = discord.utils.get(ctx.guild.roles, name="Muted")
    if not role:
        # Créer un rôle "Muted" s'il n'existe pas
        role = await ctx.guild.create_role(name="Muted")
        for channel in ctx.guild.channels:
            await channel.set_permissions(role, send_messages=False, speak=False)
    
    await member.add_roles(role, reason=reason)
    await ctx.send(f"{member} a été mute pour {duration} secondes pour : {reason}")
    
    # Unmute après la durée
    await asyncio.sleep(duration)
    await member.remove_roles(role)
    await ctx.send(f"{member} n'est plus mute.")

# Exclure un utilisateur
@bot.command(name='kick')
@commands.has_permissions(kick_members=True)
async def kick(ctx, member: discord.Member, *, reason="Aucune raison spécifiée"):
    await member.kick(reason=reason)
    await ctx.send(f"{member} a été exclu pour : {reason}")

# Ban temporaire
@bot.command(name='bantemp')
@commands.has_permissions(ban_members=True)
async def tempban(ctx, member: discord.Member, duration: int, *, reason="Aucune raison spécifiée"):
    await member.ban(reason=reason)
    await ctx.send(f"{member} a été banni pour {duration} secondes pour : {reason}")
    
    # Débannir après la durée
    await asyncio.sleep(duration)
    await ctx.guild.unban(member)
    await ctx.send(f"{member} a été débanni après {duration} secondes.")

# Donner un rôle
@bot.command(name='addrole')
@commands.has_permissions(manage_roles=True)
async def addrole(ctx, member: discord.Member, role: discord.Role):
    await member.add_roles(role)
    await ctx.send(f"Le rôle {role.name} a été ajouté à {member}.")

# Enlever un rôle
@bot.command(name='removerole')
@commands.has_permissions(manage_roles=True)
async def removerole(ctx, member: discord.Member, role: discord.Role):
    await member.remove_roles(role)
    await ctx.send(f"Le rôle {role.name} a été retiré de {member}.")

# Gérer les erreurs
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send("Vous n'avez pas les permissions nécessaires pour exécuter cette commande.")
    elif isinstance(error, commands.MissingRequiredArgument):
        await ctx.send("Argument manquant pour cette commande.")
    elif isinstance(error, commands.CommandInvokeError):
        await ctx.send("Une erreur s'est produite en exécutant cette commande.")
    else:
        await ctx.send("Une erreur inconnue s'est produite.")

@bot.event
async def on_member_join(member):
    role_name = "Membre"  # Nom du rôle à attribuer
    role = discord.utils.get(member.guild.roles, name=role_name)
    
    if role:
        try:
            await member.add_roles(role)
            print(f"Rôle {role_name} attribué à {member.name}")
        except discord.Forbidden:
            print("Le bot n'a pas les permissions pour attribuer ce rôle.")
        except discord.HTTPException as e:
            print(f"Erreur HTTP : {e}")
    else:
        print(f"Le rôle {role_name} n'existe pas.")

# Commande pour attribuer un rôle sur demande
@bot.command()
async def autorole(ctx, member: discord.Member, role_name: str):
    role = discord.utils.get(ctx.guild.roles, name=role_name)
    if role:
        try:
            await member.add_roles(role)
            await ctx.send(f"Le rôle {role_name} a été attribué à {member.mention}")
        except discord.Forbidden:
            await ctx.send("Je n'ai pas la permission d'attribuer ce rôle.")
        except discord.HTTPException as e:
            await ctx.send(f"Une erreur est survenue : {e}")
    else:
        await ctx.send(f"Le rôle {role_name} n'existe pas.")

@bot.command(name='suppsalons')
@commands.has_permissions(administrator=True)  # Limite la commande aux administrateurs
async def supprimer_salons(ctx):
    guild = ctx.guild
    await ctx.send("Suppression de tous les salons...")
    for channel in guild.channels:
        try:
            await channel.delete()
            print(f"Salon supprimé : {channel.name}")
        except Exception as e:
            print(f"Erreur lors de la suppression de {channel.name} : {e}")
    await ctx.send("Tous les salons ont été supprimés!")

@bot.command(name='suppallroles')
@commands.has_permissions(administrator=True)  # Limite la commande aux administrateurs
async def supprimer_roles(ctx):
    guild = ctx.guild
    await ctx.send("Suppression de tous les rôles...")
    for role in guild.roles:
        try:
            # Ignorer les rôles spéciaux (comme @everyone)
            if role.is_default():
                continue
            await role.delete()
            print(f"Rôle supprimé : {role.name}")
        except Exception as e:
            print(f"Erreur lors de la suppression de {role.name} : {e}")
    await ctx.send("Tous les rôles ont été supprimés!")

@bot.command(name='roleall')
@commands.has_permissions(manage_roles=True)  # Vérifie si l'utilisateur a les permissions nécessaires
async def give_role_to_all(ctx, role_id: int):
    """
    Donne un rôle à tous les membres du serveur en utilisant l'ID du rôle.
    Utilisation : !give_role_to_all <role_id>
    """
    # Récupérer le rôle à partir de l'ID
    role = ctx.guild.get_role(role_id)
    if role is None:
        await ctx.send("Rôle introuvable. Assurez-vous que l'ID est correct.")
        return

    # Vérifiez que le bot a la permission d'attribuer des rôles
    if ctx.guild.me.top_role <= role:
        await ctx.send("Je ne peux pas attribuer ce rôle car il est plus haut ou égal à mon rôle dans la hiérarchie.")
        return

    count = 0  # Compteur pour suivre le nombre de membres ayant reçu le rôle
    for member in ctx.guild.members:
        if role not in member.roles:  # Si le membre n'a pas déjà ce rôle
            try:
                await member.add_roles(role)
                count += 1
            except discord.Forbidden:
                await ctx.send(f"Impossible d'ajouter le rôle à {member.name}. Vérifiez mes permissions.")
            except Exception as e:
                await ctx.send(f"Une erreur est survenue : {e}")

    await ctx.send(f"Le rôle {role.name} a été attribué à {count} membres.")

@bot.command(name='spam')
async def spam(ctx, *, message: str):
    # Vérifie si la commande est exécutée dans un serveur
    if ctx.guild:
        for _ in range(10):  # Change ce nombre pour ajuster le spam
            await ctx.send(f"@everyone {message}")
    else:
        await ctx.send("Cette commande doit être exécutée dans un serveur.")
  
bot.run(TOKEN)
