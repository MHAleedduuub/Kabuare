require('dotenv').config();
const { Client, GatewayIntentBits, SlashCommandBuilder, EmbedBuilder, PermissionsBitField, REST, Routes } = require('discord.js');
const mongoose = require('mongoose');

// ========== MongoDB Connection ==========
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/minecraft-verify')
    .then(() => console.log('✅ Connected to MongoDB'))
    .catch(err => console.error('❌ MongoDB connection error:', err));

// ========== MongoDB Schemas ==========
const userSchema = new mongoose.Schema({
    discordId: { type: String, unique: true },
    discordUsername: String,
    minecraftUsername: String,
    verified: { type: Boolean, default: false },
    verificationCode: String,
    verifiedAt: Date,
    createdAt: { type: Date, default: Date.now }
});

const settingsSchema = new mongoose.Schema({
    guildId: { type: String, unique: true },
    logChannelId: String,
    verifyRoleId: String,
    minecraftServer: { type: String, default: 'mc.hypixel.net' }
});

const User = mongoose.model('User', userSchema);
const Settings = mongoose.model('Settings', settingsSchema);

// ========== Discord Bot Client ==========
const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
        GatewayIntentBits.GuildMembers
    ]
});

// ========== Slash Commands ==========
const commands = [
    {
        name: 'link',
        description: '🔗 Get the verification website link'
    },
    {
        name: 'setchannel',
        description: '📋 Set the verification log channel (Admin only)',
        options: [
            {
                name: 'channel',
                description: 'The channel for verification logs',
                type: 7, // CHANNEL
                required: true
            }
        ],
        default_member_permissions: '8' // Administrator
    },
    {
        name: 'setrole',
        description: '👑 Set the verified role (Admin only)',
        options: [
            {
                name: 'role',
                description: 'The role to assign after verification',
                type: 8, // ROLE
                required: true
            }
        ],
        default_member_permissions: '8'
    },
    {
        name: 'setserver',
        description: '🎮 Set the Minecraft server IP (Admin only)',
        options: [
            {
                name: 'server',
                description: 'The Minecraft server IP',
                type: 3, // STRING
                required: true
            }
        ],
        default_member_permissions: '8'
    },
    {
        name: 'settings',
        description: '⚙️ View current settings (Admin only)',
        default_member_permissions: '8'
    },
    {
        name: 'verify',
        description: '✅ Verify a user manually (Admin only)',
        options: [
            {
                name: 'user',
                description: 'The user to verify',
                type: 6, // USER
                required: true
            }
        ],
        default_member_permissions: '8'
    },
    {
        name: 'unverify',
        description: '❌ Remove verification from a user (Admin only)',
        options: [
            {
                name: 'user',
                description: 'The user to unverify',
                type: 6,
                required: true
            }
        ],
        default_member_permissions: '8'
    },
    {
        name: 'check',
        description: '🔍 Check verification status of a user',
        options: [
            {
                name: 'user',
                description: 'The user to check',
                type: 6,
                required: false
            }
        ]
    },
    {
        name: 'leaderboard',
        description: '🏆 Show top verified users'
    }
];

// ========== Bot Ready Event ==========
client.once('ready', async () => {
    console.log(`✅ Bot logged in as ${client.user.tag}`);
    
    // Register slash commands
    const rest = new REST({ version: '10' }).setToken(process.env.DISCORD_TOKEN);
    
    try {
        console.log('🔄 Registering slash commands...');
        
        await rest.put(
            Routes.applicationCommands(client.user.id),
            { body: commands }
        );
        
        console.log('✅ Slash commands registered successfully!');
    } catch (error) {
        console.error('❌ Error registering commands:', error);
    }

    // Bot status
    client.user.setPresence({
        activities: [{ 
            name: `${process.env.WEBSITE_URL || 'verification'}`,
            type: 3 // WATCHING
        }],
        status: 'online'
    });
});

// ========== Command Handler ==========
client.on('interactionCreate', async interaction => {
    if (!interaction.isCommand()) return;

    const { commandName } = interaction;

    try {
        switch (commandName) {
            case 'link':
                await handleLinkCommand(interaction);
                break;
            
            case 'setchannel':
                await handleSetChannelCommand(interaction);
                break;
            
            case 'setrole':
                await handleSetRoleCommand(interaction);
                break;
            
            case 'setserver':
                await handleSetServerCommand(interaction);
                break;
            
            case 'settings':
                await handleSettingsCommand(interaction);
                break;
            
            case 'verify':
                await handleVerifyCommand(interaction);
                break;
            
            case 'unverify':
                await handleUnverifyCommand(interaction);
                break;
            
            case 'check':
                await handleCheckCommand(interaction);
                break;
            
            case 'leaderboard':
                await handleLeaderboardCommand(interaction);
                break;
            
            default:
                await interaction.reply({ 
                    content: '❌ Unknown command!', 
                    ephemeral: true 
                });
        }
    } catch (error) {
        console.error('Command error:', error);
        await interaction.reply({ 
            content: '❌ An error occurred while executing this command.', 
            ephemeral: true 
        });
    }
});

// ========== Command Functions ==========
async function handleLinkCommand(interaction) {
    const embed = new EmbedBuilder()
        .setTitle('🔗 Minecraft Verification')
        .setDescription(`Verify your Minecraft account by visiting our website!`)
        .addFields(
            { name: 'Website URL', value: process.env.WEBSITE_URL || 'Not configured' },
            { name: 'How to verify', value: '1. Visit the website\n2. Login with Discord\n3. Enter your Minecraft username\n4. Wait for admin approval' }
        )
        .setColor('#4CAF50')
        .setTimestamp();

    await interaction.reply({ embeds: [embed] });
}

async function handleSetChannelCommand(interaction) {
    const channel = interaction.options.getChannel('channel');
    
    await Settings.findOneAndUpdate(
        { guildId: interaction.guildId },
        { logChannelId: channel.id },
        { upsert: true, new: true }
    );

    await interaction.reply({ 
        content: `✅ Verification log channel set to ${channel}`, 
        ephemeral: true 
    });
}

async function handleSetRoleCommand(interaction) {
    const role = interaction.options.getRole('role');
    
    await Settings.findOneAndUpdate(
        { guildId: interaction.guildId },
        { verifyRoleId: role.id },
        { upsert: true, new: true }
    );

    await interaction.reply({ 
        content: `✅ Verified role set to ${role.name}`, 
        ephemeral: true 
    });
}

async function handleSetServerCommand(interaction) {
    const server = interaction.options.getString('server');
    
    await Settings.findOneAndUpdate(
        { guildId: interaction.guildId },
        { minecraftServer: server },
        { upsert: true, new: true }
    );

    await interaction.reply({ 
        content: `✅ Minecraft server set to \`${server}\``, 
        ephemeral: true 
    });
}

async function handleSettingsCommand(interaction) {
    const settings = await Settings.findOne({ guildId: interaction.guildId });
    
    if (!settings) {
        return interaction.reply({ 
            content: '❌ No settings configured yet. Use the setup commands first!', 
            ephemeral: true 
        });
    }

    const embed = new EmbedBuilder()
        .setTitle('⚙️ Bot Settings')
        .setColor('#2196F3')
        .addFields(
            { name: 'Log Channel', value: settings.logChannelId ? `<#${settings.logChannelId}>` : 'Not set', inline: true },
            { name: 'Verify Role', value: settings.verifyRoleId ? `<@&${settings.verifyRoleId}>` : 'Not set', inline: true },
            { name: 'Minecraft Server', value: settings.minecraftServer || 'Not set', inline: true },
            { name: 'Website URL', value: process.env.WEBSITE_URL || 'Not configured' }
        )
        .setTimestamp();

    await interaction.reply({ embeds: [embed], ephemeral: true });
}

async function handleVerifyCommand(interaction) {
    const targetUser = interaction.options.getUser('user');
    
    // Check if user exists in database
    const user = await User.findOne({ discordId: targetUser.id });
    
    if (!user) {
        return interaction.reply({ 
            content: '❌ This user hasn\'t started the verification process yet.', 
            ephemeral: true 
        });
    }

    if (user.verified) {
        return interaction.reply({ 
            content: '❌ This user is already verified.', 
            ephemeral: true 
        });
    }

    // Update user verification status
    user.verified = true;
    user.verifiedAt = new Date();
    await user.save();

    // Assign role if configured
    const settings = await Settings.findOne({ guildId: interaction.guildId });
    
    if (settings && settings.verifyRoleId) {
        try {
            const member = await interaction.guild.members.fetch(targetUser.id);
            await member.roles.add(settings.verifyRoleId);
        } catch (error) {
            console.error('Error assigning role:', error);
        }
    }

    // Log verification
    await logVerification(interaction, targetUser, user, 'verified');

    const embed = new EmbedBuilder()
        .setTitle('✅ User Verified')
        .setDescription(`${targetUser} has been verified!`)
        .addFields(
            { name: 'Minecraft Username', value: user.minecraftUsername || 'Unknown' },
            { name: 'Verified by', value: interaction.user.toString() }
        )
        .setColor('#4CAF50')
        .setTimestamp();

    await interaction.reply({ embeds: [embed] });
}

async function handleUnverifyCommand(interaction) {
    const targetUser = interaction.options.getUser('user');
    
    const user = await User.findOne({ discordId: targetUser.id });
    
    if (!user || !user.verified) {
        return interaction.reply({ 
            content: '❌ This user is not verified.', 
            ephemeral: true 
        });
    }

    // Remove verification
    user.verified = false;
    user.verifiedAt = null;
    await user.save();

    // Remove role if configured
    const settings = await Settings.findOne({ guildId: interaction.guildId });
    
    if (settings && settings.verifyRoleId) {
        try {
            const member = await interaction.guild.members.fetch(targetUser.id);
            await member.roles.remove(settings.verifyRoleId);
        } catch (error) {
            console.error('Error removing role:', error);
        }
    }

    // Log unverification
    await logVerification(interaction, targetUser, user, 'unverified');

    const embed = new EmbedBuilder()
        .setTitle('❌ User Unverified')
        .setDescription(`${targetUser} has been unverified.`)
        .addFields(
            { name: 'Minecraft Username', value: user.minecraftUsername || 'Unknown' },
            { name: 'Unverified by', value: interaction.user.toString() }
        )
        .setColor('#FF4444')
        .setTimestamp();

    await interaction.reply({ embeds: [embed] });
}

async function handleCheckCommand(interaction) {
    const targetUser = interaction.options.getUser('user') || interaction.user;
    
    const user = await User.findOne({ discordId: targetUser.id });
    
    if (!user) {
        return interaction.reply({ 
            content: '❌ No verification data found for this user.', 
            ephemeral: true 
        });
    }

    const embed = new EmbedBuilder()
        .setTitle(`🔍 Verification Status - ${targetUser.username}`)
        .setColor(user.verified ? '#4CAF50' : '#FF4444')
        .addFields(
            { name: 'Status', value: user.verified ? '✅ Verified' : '❌ Not Verified', inline: true },
            { name: 'Minecraft Username', value: user.minecraftUsername || 'Not set', inline: true },
            { name: 'Verified At', value: user.verifiedAt ? user.verifiedAt.toLocaleDateString() : 'N/A', inline: true }
        )
        .setThumbnail(targetUser.displayAvatarURL())
        .setTimestamp();

    await interaction.reply({ embeds: [embed] });
}

async function handleLeaderboardCommand(interaction) {
    const users = await User.find({ verified: true })
        .sort({ verifiedAt: -1 })
        .limit(10);

    if (users.length === 0) {
        return interaction.reply({ 
            content: '🏆 No verified users yet!', 
            ephemeral: true 
        });
    }

    const leaderboardText = users.map((user, index) => {
        const medal = index === 0 ? '🥇' : index === 1 ? '🥈' : index === 2 ? '🥉' : '📊';
        return `${medal} **${index + 1}.** <@${user.discordId}> - \`${user.minecraftUsername}\``;
    }).join('\n');

    const embed = new EmbedBuilder()
        .setTitle('🏆 Verification Leaderboard')
        .setDescription(leaderboardText)
        .setColor('#FFD700')
        .setTimestamp();

    await interaction.reply({ embeds: [embed] });
}

// ========== Log Verification Function ==========
async function logVerification(interaction, targetUser, userData, action) {
    const settings = await Settings.findOne({ guildId: interaction.guildId });
    
    if (!settings || !settings.logChannelId) return;

    const channel = interaction.guild.channels.cache.get(settings.logChannelId);
    if (!channel) return;

    const embed = new EmbedBuilder()
        .setTitle(action === 'verified' ? '✅ User Verified' : '❌ User Unverified')
        .setColor(action === 'verified' ? '#4CAF50' : '#FF4444')
        .addFields(
            { name: 'Discord User', value: `${targetUser.tag} (${targetUser.id})`, inline: true },
            { name: 'Minecraft Username', value: userData.minecraftUsername || 'Unknown', inline: true },
            { name: 'Action by', value: interaction.user.toString(), inline: true }
        )
        .setThumbnail(targetUser.displayAvatarURL())
        .setTimestamp();

    await channel.send({ embeds: [embed] });
}

// ========== Error Handling ==========
process.on('unhandledRejection', error => {
    console.error('Unhandled promise rejection:', error);
});

client.on('error', error => {
    console.error('Discord client error:', error);
});

// ========== Login Bot ==========
client.login(process.env.DISCORD_TOKEN);
