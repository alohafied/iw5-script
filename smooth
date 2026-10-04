#include <stdinc.hpp>
#include "loader/component_loader.hpp"

#include "game/game.hpp"

namespace weapon_state
{
	using method_t = void(*)(game::scr_entref_t);

	constexpr unsigned int GETVIEWMODEL_ID = 0x82B1;
	constexpr unsigned int METHOD_BASE = 0x8000;

	constexpr std::size_t WEAP_ANIM_RIGHT   = 0x23C;
	constexpr std::size_t WEAPON_TIME_RIGHT = 0x240;
	constexpr std::size_t WEAPON_IDLE_TIME  = 0x4D4;

	method_t original_getviewmodel = nullptr;

	int get_int(unsigned int index)
	{
		if (index >= game::scr_VmPub->outparamcount)
			return 0;

		auto* value = game::scr_VmPub->top - index;

		if (value->type != game::SCRIPT_INTEGER)
			return 0;

		return value->u.intValue;
	}

	const char* get_string(unsigned int index)
	{
		if (index >= game::scr_VmPub->outparamcount)
			return nullptr;

		auto* value = game::scr_VmPub->top - index;

		if (value->type != game::SCRIPT_STRING)
			return nullptr;

		return game::SL_ConvertToString(value->u.stringValue);
	}

	unsigned char* get_player_state(game::scr_entref_t entref)
	{
		if (entref.classnum != 0)
			return nullptr;

		if (entref.entnum >= 18)
			return nullptr;

		auto* client = game::g_entities[entref.entnum].client;

		if (!client)
			return nullptr;

		// playerState_s is the first member of gclient_s.
		return reinterpret_cast<unsigned char*>(client);
	}

	void getviewmodel_hook(game::scr_entref_t entref)
	{
		const char* command = get_string(0);

		if (command)
		{
			auto* ps = get_player_state(entref);

			if (ps)
			{
				if (!_stricmp(command, "setweaponanim"))
				{
					*reinterpret_cast<int*>(ps + WEAP_ANIM_RIGHT) = get_int(1);
					return;
				}

				if (!_stricmp(command, "setweaponanimtime"))
				{
					*reinterpret_cast<int*>(ps + WEAPON_TIME_RIGHT) = get_int(1);
					return;
				}

				if (!_stricmp(command, "setweaponidletime"))
				{
					*reinterpret_cast<int*>(ps + WEAPON_IDLE_TIME) = get_int(1);
					return;
				}
			}
		}

		if (original_getviewmodel)
			original_getviewmodel(entref);
	}

	class component final : public component_interface
	{
	public:
		void post_unpack() override
		{
			auto* methods =
				reinterpret_cast<method_t*>(game::plutonium::method_table.get());

			const auto index = GETVIEWMODEL_ID - METHOD_BASE;

			original_getviewmodel = methods[index];
			methods[index] = getviewmodel_hook;
		}

		void pre_destroy() override
		{
			if (!original_getviewmodel)
				return;

			auto* methods =
				reinterpret_cast<method_t*>(game::plutonium::method_table.get());

			methods[GETVIEWMODEL_ID - METHOD_BASE] = original_getviewmodel;
		}
	};
}

REGISTER_COMPONENT(weapon_state::component)
